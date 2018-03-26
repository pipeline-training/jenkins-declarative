# Exercise 1.0
In this exercise you will setup a work environment for the lessons provided
in this workshop.  Ask the instructor for the URL of the server you will be using during the workshop.

1. Goto to the Workshop URL provided by the instructor;
2. Click on the **Create an account** link in the middle of the page under the **Login** button.
3. Complete the **Sign up** form (all fields are required) and click the **Sign up** button;
4. You should see a **Success** page - click on **the top page** link;
5. Click on the link for the **workshop** folder;
6. In the left menu, click on the **New dev-folder** link;
7. For **Enter an item name** enter your workshop **Username**, select **dev-folder** and click the **OK** button;
8. On the next screen, click the **Save** button;
9. You will now be in your very own **dev-folder** where you will be able to create and edit Pipelines for this workshop.

# Exercise 1.1

In **Exercise 1.1** we will create a simple declarative pipeline directly within the Jenkins interface.

Using the personal folder created for you in Exercise 1.0, do the following:

1. Click on the **create new jobs** link.  Alternatively you can create a job by selecting **New Item** from the left menu and then step 2 below.
2. Type **SimplePipeline** into the **Enter an item name** text box, click on **Pipeline**, and then click on the **OK** button.
3. Copy and paste the following code into the **Pipeline Script** text box near the bottom of the page:

```
pipeline {
   agent any
    
   stages {
      stage('Say Hello') {
         steps {
            echo 'Hello World!'   
         }
      }
   }
}
```

4. Click on **Save** and then click on **Build Now** in the left menu to run your pipeline.

# Exercise 1.2

In **Exercise 1.2** we will update the pipeline we created in Exercise 1.1 to execute in a docker container. To update the pipeline:

1. In the `steps` block add the following after the `echo` step:

```
  sh 'make --version'
```

2. Execute your job by clicking on **Build Now** and check the Console Log. The build will fail with `make: not found`

3. Click on configure and update the ```agent``` portion of the pipeline to read:

```
   agent {
      docker { 
        image 'gcc:latest'
        label 'docker-cloud' 
      }
   }
```

4. Execute your job by clicking on **Build Now** and check the Console Log to see how Jenkins pulls the appropriate docker image and runs your build inside the container created from that image. 

5. Remove the `label 'docker-cloud'` line from your pipeline job and build the job again by clicking on **Build Now**.  The job should still work... but why?? Pipeline Model Definition is why.  The Pipeline Model Definition allows an administrator to define the default label a job should use if one is not specified with docker.  In this case the Pipeline Model Definition is already set to `docker-cloud`.  This means that you will get an agent running dockerd everytime you use the docker {} block so using label parameter is redundant unless you need a specific docker agent.

Before going on to the next exercise let's revert our pipeline to using:

```
   agent any
```

And remove the `sh 'make --version'` step

# Exercise 1.3

For **Exercise 1.3** we are going to update our **SimplePipeline** job to demonstrate how to use environmental variables.

At the top of the pipeline insert the following code between the ```agent``` and ```stages``` blocks:  

```
   environment {
      MY_NAME = 'Mary'
   }
```

Then update the ```echo 'Hello World!'``` line to read ```echo "Hello ${MY_NAME}!"``` and run your build again to view the results.  Notice the change from '' to "".  Using double quotes will trigger extrapolation of environment variables.

We can also use environmental variables to import credentials. To demonstrate we will add the following line to our ```environment``` block:

```TEST_USER = credentials('test-user')```

We will also add the following ```echo``` steps within the ```steps``` of the Say Hello ```stage```:

```
            echo "${TEST_USER_USR}"
            echo "${TEST_USER_PSW}"
```

**Note**: After executing the build look at the console output and make note of the fact that the credential user name and password are masked when output via the echo command.

# Exercise 1.4

In **Exercise 1.4** we will alter our pipeline to accept external input in the form of a Parameter.

At the top of your pipeline insert the following block of code between the ```environment``` and ```stages``` blocks:

```
   parameters {
      string(name: 'Name', defaultValue: 'whoever you are', 
	     description: 'Who should I say hi to?')
   }
```
Then update the ```echo "Hello ${MY_NAME}!'``` line to read ```echo "Hello ${params.Name}!"``` and run your build again to view the results.

**Note**: Jenkins UI won't update properly when you save the pipeline to show the ```Build with parameters``` option so you need to run a build, view the results, and then return to the project to see the updated option.

# Exercise 1.5

For **Exercise 1.5** we are going to add a new stage after the **Say Hello** stage that will demonstrate how to ask interactively for user input. 

**Important Note** The following code demonstrates a new set of features added to Declarative Pipeline in Version 1.2.6:

  - Add options {} for stage - supports block-scoped "wrappers" like timeout and Declarative options like skipDefaultCheckout
  - Add input {} directive for stage - runs the input step with the supplied configuration before entering the when or agent for a stage, and makes any parameters provided as part of the input step available as environment variables.

The declarative `input` directive blocks the `stage` from executing and acquiring an agent - this is an important enhancement as previously a more complicated work-around was required to not tie up an agent with an input step. If the `input` is approved, the stage will then continue.

Insert the following `stage` block into your pipeline after `stage('Say Hello') {} block:

```
    stage('Deploy') {
      input {
        message "Should we continue?"
      }
      steps {
        echo "Continuing with deployment"
      }
    }
```

**Note**: To keep Jenkins from waiting indefinitely for a user response your should set a ```timeout``` for the `stage` like shown below:

```
    stage('Deploy') {
      options {
        timeout(time: 1, unit: 'MINUTES') 
      }
      input {
        message "Should we continue?"
      }
      steps {
        echo "Continuing with deployment"
      }
    }
```

# Exercise 1.6

In this example we will replace the **Deploy** stage with an input that returns data to the pipeline for use later in a subsequent step or stage.  This form of input is useful when needing to query users for additional data before continuing pipeline processing.

Replace the **Deploy** stage with the following and rerun the job:
```
    stage('Deploy') {
      input {
        message "Which Version?"
        ok "Deploy"
        parameters {
            choice(name: 'APP_VERSION', choices: "v1.1\nv1.2\nv1.3", description: 'What to deploy?')
        }
      }
      steps {
        echo "Deploying ${APP_VERSION}."
      }
    }
```


# Exercise 1.7

What happens if your input step times out? **Post Actions** are designed to handle a variety of conditions (not only failures) that could occur outside the standard pipeline flow.

In this example we will add a Post Action to our **Deploy** stage to handle a time out (aborted run). Modify your **Deploy** stage to look like:

```
    stage('Deploy') {
      options {
        timeout(time: 1, unit: 'MINUTES') 
      }
      input {
        message "Which Version?"
        ok "Deploy"
        parameters {
            choice(name: 'APP_VERSION', choices: "v1.1\nv1.2\nv1.3", description: 'What to deploy?')
        }
      }
      steps {
        echo "Deploying ${APP_VERSION}."
      }
      post {
        aborted {
          echo 'Why didn\'t you push my button?'
        }
      }
    }
      
```

On the next build wait for the input time and you will see the following line in your console output: ```Why didn't you push my button?```.

**Note**: After completing this exercise remove the ```Deploy``` stage from your pipeline so that you will not have to manually approve it each time it runs.

# Exercise 1.8

In this exercise we will combine the simplicity of declarative pipeline with
more advanced features of pipeline available via the `script {}` block.  

Scripted Pipelines can be very advanced and contain advanced flow control and variable assignment that are not available in declarative pipelines without using a `script` block.  A `script` block allows you to insert the more advanced scripted version of pipeline into a declarative pipeline.

After removing the ```stage('Deploy')``` block from the previous example add two new stages called ```stage('Get Kernel')``` and ```stage('Say Kernel')``` to your pipeline after the **Say Hello** stage:

```
    stage('Get Kernel') {
      steps {
        script {
          try {
            KERNEL_VERSION = sh (script: "uname -r", returnStdout: true)
          } catch(err) {
            echo "CAUGHT ERROR: ${err}"
            throw err
          }
        }
      }
    }
    stage('Say Kernel') {
      steps {
        echo "${KERNEL_VERSION}"
      }
    }

```
**Note:** Notice the use of the `script` block in the above example.  This allows you to use more advanced scripting options in your declarative pipelines.

**Note**: After completing this exercise remove the ```Get Kernel``` and ```Say Kernel` stages from your pipeline.


# Exercise 1.9

In this exercise we are going to add another stage to our pipeline that runs two steps in parallel on two different docker based agents (one running Java 7 and one running Java 8). The following code also includes ```sleep``` steps to demonstrate what happens when parallel steps complete execution at different times:

**Important Note** The following code demonstrates a new set of features added to Declarative Pipeline in Version 1.2 (parallel stages) and 1.2.1 (failFast inside of a parallel stage).

Add the following stage after ```stage('Say Hello')```:

```
      stage('Testing') {
        parallel {
          stage('Java 7') {
            agent { docker 'openjdk:7-jdk-alpine' }
            failFast true
            steps {
              sh 'java -version'
              sleep time: 1, unit: 'MINUTES'
            }
          }
          stage('Java 8') {
            agent { docker 'openjdk:8-jdk-alpine' }
            steps {
              sh 'java -version'
              sleep time: 2, unit: 'MINUTES'
            }
          }
        }
      }
```

**Note**: If your build breaks double check your pipeline script to make sure that the agent at the top of of the pipeline was reverted back to ```agent any``` as described in exercise 1.2.

# Exercise 1.10

In **Exercise 1.9** we are going to add a stage to our pipeline that uses a **Shared Library** to import functionality that allows us to say hi.

More information on using Shared Libraries is available here: https://jenkins.io/doc/book/pipeline/shared-libraries/

Add the following line at the top of your pipeline **above** the ```pipeline``` line:

```library 'SharedLibs'```

Then add the following stage after the **Deploy** stage:

```
      stage('Shared Lib') {
         steps {
             helloWorld("Jenkins")
         }
      }
```

The ```helloWorld``` function we are calling can be seen at: https://github.com/PipelineHandsOn/shared-libraries/blob/master/vars/helloWorld.groovy

# Exercise 1.11

Finally we will use the Blue Ocean Pipeline Editor to create a simple declarative pipeline using the following steps:

1. Click on the **Open Blue Ocean** button in the left side navigation bar
2. Click on the **New Pipeline** button
3. Click on one of the options in the **Where do you store your code?** section (Github for this course)
4. Enter your **Github token**
5. Select the **Organization** in which the repository that you want to create the Jenkinsfile in exists
6. Select **New Pipeline**
7. Choose the **Repository**
8. Click on **Create Pipeline**

Once the pipeline has created Blue Ocean will open the editor screen. We will create a few simple steps using the following instructions (feel free to veer of course and try all of the options available):

1. Click on the **+** icon next to the pipeline's **Start** node
2. Click into **Name your stage** and enter a name
3. Click on **+ Add step**
4. Click on **Shell script**
5. Type ```mvn -v``` into the text box
6. Click on **Save** to save the pipeline and execute it
7. Enter a commit message into the **Save Pipeline** pop up and click **Save & Run**

After your pipeline executes you can click on the **pencil** icon to continue editing your pipeline.
