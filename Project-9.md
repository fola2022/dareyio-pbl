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
<img width="929" alt="webhook added" src="https://user-images.githubusercontent.com/112771723/194881623-abe69bba-4180-46a2-952f-70cdb8e14bdc.png">
    
#### In Jenkins console, a freestyle project was created and configured by linking the URL of the github repository to Jenkins so that Jenkins could access the files in the repository    
 ### Running the build using 'Build Now' in Jenkins. The console output was not successfulat at first but i figured out the error. The issue was from my configuration on the Jenkins, the Github branch was in master, so i changed it to main. The first picture below is the #1 build (failed). The second picture is the successful build.
<img width="943" alt="jenskin item build" src="https://user-images.githubusercontent.com/112771723/194882020-ee47a6ee-cf25-47ec-a436-e0a596d70c88.png">
<img width="344" alt="jenkinsandgit" src="https://user-images.githubusercontent.com/112771723/194884364-a7e8960f-d710-4543-a307-8a2b1ba5025b.png">

#### The project was configured by adding two more configuration, which are Github hook trigger for GITScm polling under "Build Triggers" and "Post-build Action" 
#### After making changes in README file
<img width="276" alt="jenkins build" src="https://user-images.githubusercontent.com/112771723/194885625-ab488599-ac5f-4ac7-a2e0-c43d9902e96d.png">
<img width="289" alt="build successful" src="https://user-images.githubusercontent.com/112771723/194885954-d9829456-8356-453a-bd4b-fc8d23442092.png">
    
 #### CONFIGURing JENKINS TO COPY FILES TO NFS SERVER VIA SSH
 #### "Publish Over SSH" plugin was installed to Jenkins
 #### The Publish over SSH plugin  was configured to be able to connect to the NFS server
 #### After adding "Send build artifacts over SSH" to the Post-build configuration, the build console output became unstable. Then, i figured i have to change the permission on the NFS server. After changing permission, the console output was successful.
<img width="487" alt="nfs connected to jenkins" src="https://user-images.githubusercontent.com/112771723/194888484-aa41a898-6ccd-4225-8e71-28225cad5138.png">
<img width="289" alt="build successful" src="https://user-images.githubusercontent.com/112771723/194888166-dddcdd3d-e7a0-4b20-b3a0-e44cadf91be7.png">
<img width="492" alt="cat readme" src="https://user-images.githubusercontent.com/112771723/194885927-5ff9d869-96cd-4777-addb-e62557182348.png">

    

    
