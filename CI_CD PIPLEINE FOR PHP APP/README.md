## SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION.

SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION<br>
As part of the ongoing infrastructure development with Ansible started from Project 11, you will be tasked to create a pipeline that simulates continuous integration and delivery. <br>
Target end to end CI/CD pipeline is represented by the diagram below. It is important to know that both Tooling and TODO Web Applications are based on an interpreted (scripting) language (PHP). It means, it can be deployed directly onto a server and will work without compiling the code to a machine language.<br>
The problem with that approach is, it would be difficult to package and version the software for different releases. And so, in this project, we will be using a different approach for releases, rather than downloading directly from git, we will be using Ansible uri module.<br>

## SET-UP<br>
This project is partly a continuation of your Ansible work, so simply add and subtract based on the new setup in this project. It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. This will be quite a lot of servers altogether (But you don’t have to create them all at once. Only create servers required for an environment you are working with at the moment. For example, when doing deployments for development, do not create servers for integration, pentest, or production yet).<br>

To get started, we will focus on these environments initially.<br>
Ci<br>
Dev<br>
Pentest<br>
Both SIT – For System Integration Testing and UAT – User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding our applications. <br>
But Pentest – For Penetration testing is where we will conduct security related tests, so some other tools and specific configurations will be needed. In some cases, it will also be used for Performance and Load testing. Otherwise, that can also be a separate environment on its own. It all depends on decisions made by the company and the team running the show.<br>

```
Ansible Inventory should look like this
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

ci inventory file

```
[jenkins]
<Jenkins-Private-IP-Address>


[nginx]
<Nginx-Private-IP-Address>


[sonarqube]
<SonarQube-Private-IP-Address>


[artifact_repository]
<Artifact_repository-Private-IP-Address>
dev Inventory file
[tooling]
<Tooling-Web-Server-Private-IP-Address>


[todo]
<Todo-Web-Server-Private-IP-Address>


[nginx]
<Nginx-Private-IP-Address>


[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python


[db]
<DB-Server-Private-IP-Address>
```

pentest inventory file

```
[pentest:children]
pentest-todo
pentest-tooling


[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>


[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

Observations:<br>
You will notice that in the pentest inventory file, we have introduced a new concept pentest:children This is because, we want to have a group called pentest which covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.<br>
The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up. Totally up to you how you want to do this. Whatever works for you is absolutely fine in this scenario.
This makes us to introduce another Ansible concept called group_vars. With group vars, we can declare and set variables for each group of servers created in the inventory file.<br>
For example, If there are variables we need to be common between both pentest-todo and pentest-tooling, rather than setting these variables in many places, we can simply use the group_vars for pentest. Since in the inventory file it has been created as pentest:children Ansible recognizes this and simply applies that variable to both children.<br>

### ENVIRONMENT SET UP FOR CI/CD PIPELINE

|CI           |DEV      |SIT      |UAT     |PENTEST  |PREPROD |PRODUCTION|
|-------------|---------|---------|--------|---------|--------|----------|
|Nginx        |Nginx    |Nginx    |Nginx   |Nginx    |Nginx   |Nginx     |
|Jenkins      |Tooling  |Tooling  |Tooling |Tooling  |Tooling |Tooling   |
|MySQL        |PHP-Todo |PHP-Todo |PHP-Todo|PHP-Todo |PHP-Todo|PHP-Todo  |
|Artifactory  |         |         |
|Sonaqube     |         |         |



Select GitHub<br>


Connect Jenkins with GitHub<br>

Login to GitHub & Generate an Access Tokenhttps://www.darey.io/wp-content/uploads/2021/07/Jenkins-Create-Access-Token-To-Github.png<br>



Copy Access Token<br>


Paste the token and connect<br>


Create a new Pipeline<br>


At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.<br>


Here is our newly created pipeline. It takes the name of your GitHub repository.<br>


Let us create our Jenkinsfile<br>

Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.<br>


Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage<br>
```
pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```
Now go back into the Ansible pipeline in Jenkins, and select configure<br>


Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile<br>


Back to the pipeline again, this time click "Build now"<br>


This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.
To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.
Click on Blue Ocean<br>


Select your project<br>

Click on the play button against the branch<br>


Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.
Let us see this in action.<br>

Create a new git branch and name it feature/jenkinspipeline-stages<br>

Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.<br>
```
   pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }


    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```
To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.<br>

Click on the "Administration" button<br>


Navigate to the Ansible project and click on "Scan repository now"<br>


Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.<br>


In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.<br>


A QUICK TASK FOR YOU!<br>

1. Create a pull request to merge the latest code into the main branch<br>

2. After merging the PR, go back into your terminal and switch into the main branch.<br>

3. Pull the latest change.<br>

4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)<br>

   1. Package <br>

   2. Deploy <br>

   3. Clean up<br>

5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch<br>

6. Eventually, your main branch should have a successful pipeline like this in blue ocean<br>



RUNNING ANSIBLE PLAYBOOK FROM JENKINS<br>

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:<br>

Installing Ansible on Jenkins<br>

Installing Ansible plugin in Jenkins UI
Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)
You can watch a 10 minutes video here to guide you through the entire setup<br>
Note: Ensure that Ansible runs against the Dev environment successfully.<br>
Possible errors to watch out for:
Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)<br>
Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.<br>
https://wiki.jenkins.io/display/JENKINS/Building+a+software+project<br>

Possible issues to watch out for when you implement this<br>
Remember that ansible.cfg must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.<br>
If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.<br>
Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.<br>
If everything goes well for you, it means, the Dev environment has an up-to-date configuration. But what if we need to deploy to other environments?
Are we going to manually update the Jenkinsfile to point inventory to those environments? such as sit, uat, pentest, etc.<br>
Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.<br>
Think about those for a minute and try to work out which one sounds more like a better solution.<br>
Manually updating the Jenkinsfile is definitely not an option. And that should be obvious to you at this point. Because we try to automate things as much as possible.<br>
Well, unfortunately, we will not be doing any of the highlighted options. What we will be doing is to parameterise the deployment. So that at the point of execution, the appropriate values are applied.<br>























ANSIBLE ROLES FOR CI ENVIRONMENT
Now go ahead and Add two more roles to ansible:
SonarQube (Scroll down to the Sonarqube section to see instructions on how to set up and configure SonarQube manually)
Artifactory
Why do we need SonarQube?
SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities. Watch a short description here. There is a lot more hands on work ahead with SonarQube and Jenkins. So, the purpose of SonarQube will be clearer to you very soon.
Why do we need Artifactory?
Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of your build process is stored. It can be used for certain other automation, but we will it strictly to manage our build artifacts.
Watch a short description here Focus more on the first 10.08 mins
Configuring Ansible For Jenkins Deployment
In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.
To do this,
Navigate to Jenkins URL
Install & Open Blue Ocean Jenkins Plugin
Create a new pipeline








