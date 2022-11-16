# EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP
[Link to the repository](https://github.com/fola2022/Ansible-Config-mgt/tree/main/roles)
#### The concept of CI/CD was implemented in this project, in which the PHP application from github are pushed to Jenkins to run a multi-branch pipeline job. Blue open plugin was installed in the Jenkins UI and use to view build jobs run on each branches of the repository simultaneously. This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job was packaged and pushed to sonarqube server for testing before it is deployed to artifactory in which ansible job is triggered to deploy the application to production environment.
<img width="907" alt="blueocean" src="https://user-images.githubusercontent.com/112771723/202173874-6fd90728-53b2-412b-8ab1-76fa61289ab8.png">

### 5 EC2 Instances was used
#### For Jenkins server (Redhat)
#### For MySql database (Redhat)
#### For SonarQube
#### For Artifactory
#### For Nginx

### STEP 1: Configuring Ansible For Jenkins Deployment
#### In order to run ansible commands from Jenkins UI, the following implementation was done
#### - Installing Blue Ocean plugin from ‘manage plugins’ on Jenkins
#### - Creating new pipeline job on the Blue Ocean UI from Github
#### - Using the generated Github access token to get access to the repository 
#### - Selecting Ansible-Config-mgt repository to create a new pipeline


