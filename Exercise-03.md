# Exercise 3.1

In this exercise we are going to quickly create a new pipeline to demonstrate how Checkpoints work and how end users can interact with Checkpoints once a job as been built. To test this create a new pipeline job in your personal folder copying and pasting the following code into the Pipeline Script textbox:

```
pipeline {
   agent none
   stages {
      stage('One') {
         agent any
         steps {
            echo 'Stage One - Step 1'
         }
      }
      stage('Checkpoint') {
         agent none
         steps {
            checkpoint 'Checkpoint'
         }
      }
      stage('Two') {
         agent any
         steps {
            echo 'Stage Two - Step 1'
         }
      }
   }
}
```

After saving the job click on **Build Now** to run it.

When the job has completed running you will see a **Resume** icon in the build's **Stage View**. Clicking on the **Resume** icon gives you the ability to:

* **Delete** - Delete the cached artifacts and configuration for that build;
* **Restart** - Restart the build from the checkpoint.

**Note**: Deleting a checkpoint doesn't make the **Resume** icon vanish.

# Exercise 3.2
In this exercise we are going to set-up two Pipeline jobs that demonstrate CloudBee's Cross Team Collaboration feature. We will need two separate Pipelines - one that publishes an event - and two - another that is triggered by an event.

### Publish Event

```
pipeline {
    agent none
    stages {
        stage('Publish Event') {
            steps {
                publishEvent(generic('beeEvent'))
            }
        }
    }
}
```

### Event Trigger

```
pipeline {
    agent none
    triggers {
        eventTrigger(event(generic('beeEvent')))
    }
    stages {
        stage('Event Trigger') {
            when {
                expression { 
                    return currentBuild.rawBuild.getCause(com.cloudbees.jenkins.plugins.pipeline.events.EventTriggerCause)
                }
            }
            steps {
                echo 'triggered by published event'
            }
        }
    }
}
```

After creating both of these Pipeline jobs you will need to run the **Event Trigger** job once so that the trigger is registered. Once that is complete, click on **Build Now** to run the **Publish Event** job. Once that job has completed, the **Event Trigger** job will be triggered after a few seconds. The logs will show that the job was triggered by an `Event Trigger` and the `when` expression will be true.

# Exercise 3.3

In the following exercise we are going to demonstrate how you can use the Custom Marker feature of CloudBees Jenkins Enterprise to assign pipeline to a job based on an arbitrary file name like pom.xml.

In order to complete the following exercise you will need to fork the following repository into the Github organization you created in **Exercise 2.1**:

* https://github.com/PipelineHandsOn/maven-project

Once that repository is forked:

1. Click on the Github organization project you created in **Exercise 2.2**.
2. Click on **Configure**
3. Under **Project Recognizers** select **Custom Script**
4. In **Marker file** type ```pom.xml```
5. Under **Pipeline** - **Definition** select **Pipeline script from SCM**
6. Select **Git** from **SCM**
7. In **Repository URL** enter: ```https://github.com/PipelineHandsOn/custom-marker-files```
8. Select your credentials from the **Credentials** menu
9. In **Script path** enter: ```Jenkins-pom```
10. Click on **Save**
11. Click on **Scan Organization Now**

When the scan is complete your **Github Organization** project should now have both the **sample-rest-server** project and the **maven-project**.
