# CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP
[Link to the repository](https://github.com/fola2022/Ansible-Config-mgt/tree/main/roles)
#### The concept of CI/CD was implemented in this project, in which the PHP application from github are pushed to Jenkins to run a multi-branch pipeline job. Blue open plugin was installed in the Jenkins UI and use to view build jobs run on each branches of the repository simultaneously. This is done in order to achieve continuous integration of codes from different developers. After which the artifacts from the build job was packaged and pushed to sonarqube server for testing before it is deployed to artifactory in which ansible job is triggered to deploy the application to production environment.
<img width="907" alt="blueocean" src="https://user-images.githubusercontent.com/112771723/202173874-6fd90728-53b2-412b-8ab1-76fa61289ab8.png">

### 5 EC2 Instances was used. They are;
##### Jenkins server (Redhat)
##### MySql database (Redhat)
##### Nginx
##### SonarQube
##### Artifactory (Ubuntu)


### STEP 1: Configuring Ansible For Jenkins Deployment
#### In order to run ansible commands from Jenkins UI, the following implementation was done
#### - Installing Blue Ocean plugin from ‘manage plugins’ on Jenkins
#### - Creating new pipeline job on the Blue Ocean UI from Github
#### - Using the generated Github access token to get access to the repository 
#### - Selecting Ansible-Config-mgt repository to create a new pipeline

### STEP 2: Jenkins installation on the Instance
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
### STEP 3: CREATING DEPLOY DIRECTORY AND BUILD ON JENKINS
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
<img width="881" alt="main build on blue ocean" src="https://user-images.githubusercontent.com/112771723/202196459-bb9134c9-5950-4f53-a465-0f5f25fbbf7b.png">

#### A new git branch "feature/jenkinspipeline-stages" was created and use for the build. 
#### The following code was added to the Jenkinsfile to create more stages and make the new branch show up in Jenkins, scan the repository was done.
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
```
#### The branch was merged with the main branch
<img width="898" alt="after merging main branch" src="https://user-images.githubusercontent.com/112771723/202203534-84a63564-e065-4df4-81fc-25dec0815a7e.png">

### More stages were added and git push was done
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

    stage('Package') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Clean up') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
  }
}
```
### Git add, commit and push was done to push the new update
```
git add .
git commit -m "commit message"
git push
```
![build](https://user-images.githubusercontent.com/112771723/202197718-de9bfb81-d7bf-44e8-951e-374d22c80798.png)

### STEP 4: RUNNING ANSIBLE PLAYBOOK FROM JENKINS
#### Step 4.1 Commands to install Ansible and it dependencies on Jenkins server and Ansible plugin in Jenkins UI
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
<img width="611" alt="postsq" src="https://user-images.githubusercontent.com/112771723/202469504-173fd418-e400-4854-b318-c97426083e33.png">

#### After installing Ansible plugin in Jenkins UI, all the codes in Jenkinsfile was deleted to start from the scratch
#### For ansible to be able to locate the roles in the playbook, the ansible.cfg filw was created in the deploy directory and then exported from the Jenkinsfile code which runs with tags 'webserver'.
<img width="928" alt="ansible cfg file" src="https://user-images.githubusercontent.com/112771723/202206106-ffa130c2-0a36-4e7a-80fc-06f108be750f.png">

#### Using the jenkins pipeline syntax Ansible tool to generate syntax for executing playbook
![pipeline syntax 1](https://user-images.githubusercontent.com/112771723/202209114-96cb953a-254e-4e8c-ab6a-01d9d76a9863.png)
![pipeline systax ansible 2](https://user-images.githubusercontent.com/112771723/202209454-b1597067-18e0-42a2-8e15-7ad0a520671e.png)
![pipeline systax ansible 3](https://user-images.githubusercontent.com/112771723/202209781-051ad4fa-2165-4673-ab14-732d2572ea27.png)

### Nginx and Mysql roles were collected from ansible galaxy and installed to the Jenkins-ansible server
<img width="772" alt="nginx installeed" src="https://user-images.githubusercontent.com/112771723/202468868-493175c0-b393-4573-b625-26becbc4917f.png">
<img width="745" alt="mysql roles installed" src="https://user-images.githubusercontent.com/112771723/202468813-dfa1c43d-4225-4eb6-a198-94164a6b31dd.png">

#### Updating Jenkinsfile to introduce parameterization
```
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev.yml',  description: 'This is the inventory file for the environment to deploy configuration')
    }

  stages {
    stage("Initial Clean Up") {
      steps {
        dir("${WORKSPACE}") {
           deleteDir()
        }
      }
    }

    stage('SCM Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/fola2022/ansible-config-mgt.git'
      }
    }
    
    stage('Execute Playbook') {
      steps {
        withEnv(['ANSIBLE_CONFIG = ${WORKSPACE}/deploy/.ansible.cfg']) {
          ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible2', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml', tags: 'webservers'
        }
      }
    }
  }
}
```
<img width="502" alt="excute unit successful" src="https://user-images.githubusercontent.com/112771723/202462050-75ffa7a3-445c-4cdc-81c4-4288a047907c.png">

### STEP 5: CI/CD PIPELINE FOR TODO APPLICATION
#### The goal here is to deploy the application onto servers directly from Artifactory rather than from git.
#### First, Artifactory role was collected from ansible galaxy and installed into the Jenkins-ansible server
<img width="550" alt="artifactory role" src="https://user-images.githubusercontent.com/112771723/202213905-ebd68cf5-cf90-42d7-a529-fc12476a7311.png">

#### Step 5.1: Preparing Jenkins
#### This repo was forked to mine
```
https://github.com/darey-devops/php-todo.git
```
#### Installed Jenkins plugins: 
#### - Plot plugin: This will use to display tests reports, and code coverage information.
#### - Artifactory plugin: This will be used to easily upload code artifacts into an Artifactory server. 
#### In the Jenkins UI, Artifactory was configured in configure system
### Installing PHP and Composer
#### Commands to install PHP and it dependencies
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

#### Step Commands to install Composer
```
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/bin/composer
```
<img width="518" alt="composer" src="https://user-images.githubusercontent.com/112771723/202185844-a593d1c2-2caa-4adf-b30f-f7d29080b93e.png">

#### Configure Artifactory in Jenkins UI
#### - Clicked Manage Jenkins, click Configure System
#### - Scroll down to JFrog, click Add Artifactory Server
#### - Enter the Server ID as "artifactory-server"
#### - Enter the URL as:
```
http://<artifactory-server-ip>:8082/artifactory
```
<img width="796" alt="artifactory on browser" src="https://user-images.githubusercontent.com/112771723/202221941-1bea7584-183e-4c95-9b68-79ecbfcd154b.png">

### Step 5.2: Integrate Artifactory repository with Jenkins
#### Created a dummy Jenkinsfile in the root of the php-todo repo and in the Blue Ocean, I created a multibranch pipeline
#### Created a database and user on the database server
```
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```
#### Database connectivity requirements was updated in the file .env.sample
<img width="182" alt="env  file " src="https://user-images.githubusercontent.com/112771723/202470274-157d62a9-5dc0-4064-adb1-473d3acdd17c.png">

#### On database server, installing mysql: `sudo yum install mysql-server`


