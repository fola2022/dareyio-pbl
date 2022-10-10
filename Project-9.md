## CONTINIOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE USING JENKINS
### STEP 1: INSTALLING AND CONFIGURING JENKINS SERVER
#### First install JDK because Jenkins is a Java-based application
##### Command:
```
sudo apt update
sudo apt install default-jdk-headless
```
<img width="519" alt="apt update" src="https://user-images.githubusercontent.com/112771723/194874769-37533e5e-7cb0-435b-91f2-6e520b0cebbf.png">
<img width="520" alt="install jdk" src="https://user-images.githubusercontent.com/112771723/194874839-e15d8c4e-0494-46b1-887a-7aa8d03a6be3.png">

#### Installing Jenkins
##### Commands:
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
<img width="534" alt="installing jenkin1 " src="https://user-images.githubusercontent.com/112771723/194875138-231ef238-c36a-459f-8163-480e3e09ab3b.png">
<img width="512" alt="jenkins installed" src="https://user-images.githubusercontent.com/112771723/194875175-4f35b5e4-8971-4e02-b560-04e89f357c37.png">

#### Making sure Jenkins is active and running
##### Command:
```
sudo systemctl status jenkins
```
<img width="515" alt="jenkins running" src="https://user-images.githubusercontent.com/112771723/194875493-37ebed9d-3383-42f4-89fe-3925dd0ca7aa.png">

#### TCP port 8080 was opened in the inbound rule because by default, Jenkins use port TCP port 8080
##### Jenkins was access on the broswer using  http://<Jenkins-Server-Public-IP-Address>:8080
##### The below code was run on the terminal in order to retrive the Jenkins password for login
##### Command: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
    
### STEP 2: CONFIGURING JENKINS TO RETRIEVE SOURCE CODES FROM GITHUB USING WEBHOOKS
#### Webhook was enabled in the github repository and the payload URL was edited with the jenkins server Public IP Address
#### In Jenkins console, a freestyle project was created and configured by linking the URL of the github repository to Jenkins so that Jenkins could access the files in the repository    
  
