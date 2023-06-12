# DEPLOYMENT AUTOMATION WITH JENKINS SERVER

One of the ways to guarantee fast and repeatable deployments is the Automation of routine tasks<br>.
In this project, we are going to start automating part of our routine tasks with a free and open-source automation server – Jenkins.<br>
It is one of the most popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".<br>

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. 
Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository<br>
In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.<br>

Task<br>
Enhance the architecture prepared in Project 8 by adding a Jenkins server, and configuring a job to automatically deploy source code changes from Git to the NFS server.<br>
Here is what your updated architecture will look like upon completion of this project:<br>


## Step 1 – INSTALL THE JENKINS SERVER
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins" <br>
- Install JDK (since Jenkins is a Java-based application)<br>
`sudo apt update`<br>
`sudo apt install default-jdk-headless`<br>
Install Jenkins<br>
`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`<br>
`sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`<br>
`sudo apt update`<br>
`sudo apt install jenkins`<br>
Make sure Jenkins is up and running<br>
`sudo systemctl status jenkins`<br>


By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 security group.<br>


Perform initial Jenkins setup.<br>
From your browser access 
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```
You will be prompted to provide a default admin password.Retrieve it from your server: <br>
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Then you will be asked which plugins to install – choose **suggested plugins.**

Once plugin installation is done – create an admin user and you will get your Jenkins server address.
The installation is completed!

## STEP 2 – CONFIGURE JENKINS TO RETRIEVE SOURCE CODES FROM GITHUB USING WEBHOOKS.<br>

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). <br>
This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.<br>

- Enable webhooks in your GitHub repository settings

- Go to Jenkins web console, click "New Item" and create a "Freestyle project"

- To connect your GitHub repository, you will need to provide its URL, you can copy it from the repository itself.

- In the configuration of your Jenkins freestyle project choose Git repository, and provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

- Save the configuration and let us try to run the build. 
For now, we can only do it manually.
Click the "Build Now" button, if you have configured everything correctly, the build will be successful and you will see it under #1

- You can open the build and check in "Console Output" if it has run successfully. If so – congratulations!<br>
You have just made your very first Jenkins build!But this build does not produce anything and it runs only when we trigger it manually.<br> 
Let us fix it.<br>

- Click "Configure" your job/project and add these two configurations.<br>
- Configure triggering the job from the GitHub webhook:<br>

- Configure "Post-build Actions" to archive all the files – files resulting from a build are called "artifacts".<br>

- Now, go ahead and make some changes in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.<br>
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on the Jenkins server.<br>


You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and file transfer is initiated by GitHub).<br> There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.<br>
By default, the artifacts are stored on the Jenkins server locally<br>
`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

## STEP 3 - CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.<br>
Jenkins is a highly extendable application and there are 1400+ plugins available. <br>
We will need a plugin that is called "Publish Over SSH".<br>

1. Install the "Publish Over SSH" plugin.
- On the main dashboard select "Manage Jenkins" and choose the "Manage Plugins" menu item.
- On the "Available" tab search for the "Publish Over SSH" plugin and install it.

2. Configure the job/project to copy artifacts over to the NFS server.
- On the main dashboard select "Manage Jenkins" and choose the "Configure System" menu item.
- Scroll down to Publish over the SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a private key (the content of .pem file that you use to connect to the NFS server via SSH/Putty)
- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since the NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server.
Test the configuration and make sure the connection returns Success. <br>
**Remember, that TCP port 22 on NFS server must be open to receive SSH connections.**


- Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action".

- Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **_**_**<br>

- Save this configuration and go ahead, and change something in README.MD file in your GitHub Tooling repository.
Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:


```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file<br>
`cat /mnt/apps/README.md`<br>
If you see the changes you had previously made in your GitHub – the job works as expected.<br>

Congratulations!You have just implemented your first Continous Integration solution using Jenkins CI. 





