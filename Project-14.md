# EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP
[Link to the repository](https://github.com/fola2022/Ansible-Config-mgt/tree/main/roles)
#### The concept of CI/CD was implemented in this project, in which the PHP application from github are pushed to Jenkins to run a multi-branch pipeline job. Blue open plugin was installed in the Jenkins UI and use to view build jobs run on each branches of the repository simultaneously. This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job was packaged and pushed to sonarqube server for testing before it is deployed to artifactory in which ansible job is triggered to deploy the application to production environment.
<img width="907" alt="blueocean" src="https://user-images.githubusercontent.com/112771723/202173874-6fd90728-53b2-412b-8ab1-76fa61289ab8.png">

### 5 EC2 Instances was used. They are;
##### Jenkins server (Redhat)
##### MySql database (Redhat)
##### Nginx
##### SonarQube
##### Artifactory


### STEP 1: Configuring Ansible For Jenkins Deployment
#### In order to run ansible commands from Jenkins UI, the following implementation was done
#### - Installing Blue Ocean plugin from ‘manage plugins’ on Jenkins
#### - Creating new pipeline job on the Blue Ocean UI from Github
#### - Using the generated Github access token to get access to the repository 
#### - Selecting Ansible-Config-mgt repository to create a new pipeline

### STEP 2: Jenkins, Ansible, PHP and Composer installation on the Instance
#### Step 2.1: Commands to install Jenkins and it dependencies
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install java-11-openjdk-devel -y
```
<img width="592" alt="installed jenkins" src="https://user-images.githubusercontent.com/112771723/202182921-f00e2612-5aa7-4c03-9cd2-af0cc1ec9294.png">
<img width="947" alt="jenkins-start-enable-running" src="https://user-images.githubusercontent.com/112771723/202183028-f2763f6e-6391-4200-9392-c353888e9af7.png">

#### Then open the bash profile
```
vi .bash_profile (add the export)
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```
#### Then reload bash bash_profile with below command
```
source ~/.bash_profile
```
```
sudo yum install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
sudo systemctl daemon-reload
```
#### Step 2.2: Commands to install Ansible and it dependencies
```
sudo yum install ansible -y
ansible --version
python3 -m pip install --upgrade setuptools
python3 -m pip install --upgrade pip
python3 -m pip install PyMySQL
python3 -m pip install mysql-connector-python
python3 -m pip install psycopg2==2.7.5 --ignore-installed
ansible-galaxy collection install community.postgresql
```
<img width="945" alt="ansible installed" src="https://user-images.githubusercontent.com/112771723/202183336-8fe64a51-6948-460e-9f39-b2248f1473db.png">
<img width="692" alt="ansible version" src="https://user-images.githubusercontent.com/112771723/202183582-261de62a-7476-432e-80d3-38e2bcff2a06.png">
<img width="947" alt="ansible collection" src="https://user-images.githubusercontent.com/112771723/202187481-70a6fc10-ebe2-4865-97f1-0049bd39b403.png">

#### Step 2.3: Commands to install PHP
```
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
```
<img width="947" alt="php install 1" src="https://user-images.githubusercontent.com/112771723/202185588-13c71402-8f43-4de1-9136-b1184129c162.png">
<img width="949" alt="php install 2" src="https://user-images.githubusercontent.com/112771723/202185653-13f77291-2065-481b-95b4-b613d1f554b1.png">
<img width="565" alt="php active and running status" src="https://user-images.githubusercontent.com/112771723/202185736-6f4c6775-7a44-418f-9b7f-6367018c4615.png">

#### Step 2.4: Commands to install Composer
```
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/bin/composer
```
<img width="518" alt="composer" src="https://user-images.githubusercontent.com/112771723/202185844-a593d1c2-2caa-4adf-b30f-f7d29080b93e.png">

### STEP 3
#### A new directory "Deploy" was created in the Ansible-config-mgt and a file named "Jenkinsfile was created in it.
```
mkdir deploy
touch Jenkinsfile
```
<img width="365" alt="mkdir deploy and file" src="https://user-images.githubusercontent.com/112771723/202186789-9e7f301d-55e4-4b65-bcd8-f6e6c356ad42.png">

#### The below code was placed in the Jenkinsfile
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
#### Step 3.1 Back to the Jenkins UI, Under Configure, Build configuration was done, entering the path to the Jenkinsfile, which is deploy/Jenkinsfile. Then a build was triggered by clicking on Build Now in the Jenkins UI.

![deploy-jenkinsfile](https://user-images.githubusercontent.com/112771723/202194600-7a5bf948-b934-4deb-ab77-2da26570d756.png)












```





