# SHELL SCRIPTING
## ONBOARDING NEW LINUX USERS ON A SERVER
### The script
![shell script](https://user-images.githubusercontent.com/112771723/192092737-967c18c9-be85-4008-959a-e794c5cc264e.png)

### Creating project folder called Shell and creating names.csv file in it. The names of the new linux users were inserted in the names.csv file. Also,  A file named onboard.sh was created in the project folder Shell and the shell script was placed in it. Files for both the private and public keys were also created.
#### Commands are;
##### mkdir Shell
##### cd Shell
##### touch names.csv
##### vi names.csv
#####  touch id_rsa.pub
##### touch id_rsa
<img width="326" alt="11" src="https://user-images.githubusercontent.com/112771723/192093194-3733372b-c4b8-4a7b-8249-b179074c38be.png">
<img width="550" alt="onboard successful 13" src="https://user-images.githubusercontent.com/112771723/192093263-38241acd-5411-4864-91b0-bb6ca9049185.png">
<img width="299" alt="user name" src="https://user-images.githubusercontent.com/112771723/192093828-c6d6d876-d972-48bd-a728-6e0f57fc18e7.png">

### The users were successfully added to the developers group and permission was alos changed
##### Commands are; sudo groupadd developers
#####               sudo chmod +x onboard.sh
#####               ls -la /home
<img width="382" alt="12" src="https://user-images.githubusercontent.com/112771723/192093315-432d5062-ee77-47b2-9726-3608ec0c6892.png">
<img width="389" alt="234" src="https://user-images.githubusercontent.com/112771723/192093338-535a6f50-e694-4b54-9b5f-ad22b81525e1.png">

### Developers id
##### Command: getent group developers
<img width="453" alt="developer id 14" src="https://user-images.githubusercontent.com/112771723/192093424-c0fe0bee-1b04-44f3-b2ee-3358619cd87f.png">

### Testing few users
<img width="490" alt="userJane" src="https://user-images.githubusercontent.com/112771723/192093679-846c7c43-cd25-47ec-b083-c0cd7a7a7030.png">
<img width="551" alt="userGael" src="https://user-images.githubusercontent.com/112771723/192093685-7a3c0525-8ea5-4bf4-a532-e5bf451ef3ca.png">
<img width="488" alt="userbisola" src="https://user-images.githubusercontent.com/112771723/192093639-43f6f0c8-b56c-4f8d-a9f2-541d36d1e8de.png">

#### Below is the link to the demo of this project
[Link](https://s3.console.aws.amazon.com/s3/object/folashade-pbl?region=us-east-1&prefix=aux-project.mp4)




