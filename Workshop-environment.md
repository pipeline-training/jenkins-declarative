# Introduction to Declarative Pipelines - Environment

This document describes how to create a shared CloudBees Jenkins Enterprise Master environment on jenkins.beedemo.net that the participants will use.

## Create the Managed Master

The following instructions can be used to provision a customized CJE Master on jenkins.beedmeo.net for the workshop:

1. Navigate to jenkins.beedemo.net (https://jenkins.beedemo.net/) and login;
2. Click on **New Item** at the top of the left menu;
3. **Enter an item name** for your workshop master ;
4. Select **Managed Master** and click the **OK** button;
5. On the Managed Master configure page:
  1. Under **Provisioning**:
    1. Select **Intro to Declarative Pipeline** for the **Docker image**
    2. Double the **Jenkins Master Memory in MB** setting from **2048** to **4096**
    3. Increase the **Jenkins Master CPUs** from **0.2** to **0.8**
  2. Uncheck the **Enabled** checkbox for **Health Reporting**
  3. Uncheck the **Enable** checkbox for **Analytics Reporting**
  4. VERY IMPORTANT - check the **Opt-out** checkbox for **Security Setting Enforcement**
6. Once the workshop Managed Master is up and running login with the **User** **admin** and provided **Password**
7. Ensure that you see the following items on the first page:
  - **dev-folder** a CloudBees Folder Template
  - **Operations Center Shared Templates** shared Docker Agent Templates from CJOC
  - **workshop** Folder - folder in which participants will be able to create individual folders

### Participant Instructions
1. Goto to the Workshop URL provided by the instructor;
2. Click on the **Create an account** link 
3. Complete the **Sign up** form (all fields are required) and click the **Sign up** button;
4. You should see a **Success** page - click on **the top page** link;
5. Click on the link for the **workshop** folder;
6. In the left menu, click on the **New dev-folder** link;
7. For **Enter an item name** enter your workshop **Username**, select **dev-folder** and click the **OK** button;
8. On the next screen, click the **Save** button;
9. You will now be in your very own **dev-folder** where you will be able to create and edit Pipelines for this workshop.

### Setup Shared Library

In exercise 1.7 of the workshop participants will use a function contained within a Global Pipeline Library. Prior to running that exercise you need to add the library to the Jenkins Master using the following steps:

1. Click on the **Manage Jenkins** link
2. Click on **Configure System**
3. Scroll down to **Global Pipeline Libraries** and click on **Add**
4. Enter the following in the form fields:
   - Name: ```SharedLibs```
   - Default version: ```master```
5. Select **Modern SCM**
6. Select **Github**
7. For **Credentials** select **Add** and fill out a new **Username with password:
	- Username: ```PipelineHandsOn```
	- Password: Your Github token
	- ID: PipelineHandsOn-ID
	- Description: can be left blank
8. Select your newly added Credential from the dropdown list
9. Enter **PipelineHandsOn** in the **Owner** text box
10. Select **shared-libraries** from the **Repository** drop down
11. Click on **Save**

### Add test-user Credential

In exercise 1.3 participants will use the ```test-user``` credential in a pipeline so it needs to be added in advance. To add the credential:

1. From the Jenkins Master home page click on **Credentials**, then click on **System**, then **Global credentials (unrestricted)**;
2. Click on **Add Credentials**;
3. In the **Add Credentials** form select **Username with password**, enter ```test-user``` for **Username**, **Password**, and the **ID** field;
4. Click on **OK** to save the new credential;

### Send a Pre-Worshop Email to Participants

Prior to the workshop you should send out an email to participants to let them know what they will need to fully participate in the workshop. Below is a sample of the content you should send:

```
Dear Intro to Declarative Pipeline Workshop Participant:

In order to follow along with the hands on portion of the workshop students should have the following resources available to them:

  * Internet access
  * An account on Github.com and a basic understanding of how to use Github and Git to do things like fork a repository, edit files in the web UI, and create pull requests
  * A personal access token for your Github account (https://github.com/PipelineHandsOn/intro-to-declarative-pipeline/blob/master/Github-Personal-Access-Token.md) with the following permissions:
    - repo: all
    - admin:repo_hook: all
    - admin:org_hook
    - user: all
  * Access to the CloudBees Jenkins Master hosted on AWS used for the workshop (link to be provided the day of the workshop)

Thanks and I look forward to meeting working with everyone soon!

[Your name here obviously]
```

### Verify the Master is Configured Properly

It is always a good idea to test the master prior to the workshop to avoid any surprises. The best way to do that is to run through some of the exercises and verify the participant user accounts.
