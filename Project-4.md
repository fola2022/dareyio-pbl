## MEAN STARCK DEPLOYMENT TO UBUNTU
### Ubuntu was first updated and upgraded
#### Commands are:
##### sudo apt update
##### sudo upgrade
##### sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
##### curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
<img width="628" alt="upgrade" src="https://user-images.githubusercontent.com/112771723/191031543-67ad3921-816b-4ebb-809b-13bb2515637c.png">
<img width="740" alt="cert  1" src="https://user-images.githubusercontent.com/112771723/191031584-d4eb1105-3294-490f-a059-8f9f2c2a3d83.png">
<img width="779" alt="cert  2" src="https://user-images.githubusercontent.com/112771723/191031616-73a6e72d-f949-4b0a-ac42-30c8446cae20.png">

### STEP 1: Installing NodeJs
#### Command: sudo apt install -y nodejs
<img width="677" alt="install node js" src="https://user-images.githubusercontent.com/112771723/191030280-d2bee9b8-1136-4240-8d88-150377147fe2.png">

### STEP 2: Installing MongoDB and starting the Server
#### Commands are:
#####  sudo apt install -y mongodb
##### sudo service mongodb start
##### sudo systemctl status mongodb
<img width="761" alt="install mongodb" src="https://user-images.githubusercontent.com/112771723/191032687-19d20aa5-b16b-4212-ab3d-3dba4f206511.png">
<img width="601" alt="mongodb service running" src="https://user-images.githubusercontent.com/112771723/191032856-8a18c8bd-c369-45f4-80c1-6440820c07b4.png">

### Installing npm Node pacakage manager, body-parser pacakge and creating a folder name Books
#### Commands are:
##### sudo apt install -y npm
##### sudo npm install body-parser
<img width="491" alt="npm" src="https://user-images.githubusercontent.com/112771723/191033686-2320c74c-6f41-47f5-bcb8-44a9e69d9bee.png">
<img width="552" alt="install body parser, mkdir book" src="https://user-images.githubusercontent.com/112771723/191035797-614d704b-2ee4-4720-a9fe-9fc6de49919a.png">

### STEP 3: INSTALLING EXPRESS AND SETTING UP ROUTES TO THE SERVER
#### Commands are:
##### sudo npm install express mongoose
##### mkdir apps && cd apps
##### vi routes.js
##### mkdir models && cd models
##### vi book.js
<img width="551" alt="npm intall express and mongos" src="https://user-images.githubusercontent.com/112771723/191038246-f7a4812b-a595-49db-9b52-afb4a9e61ebc.png">
<img width="443" alt="routes and book js" src="https://user-images.githubusercontent.com/112771723/191038433-d07d9901-5af7-4925-a757-25b8fa7a0cd4.png">

### STEP 4: Access the routes with AngularJS
#### Commands are:
##### mkdir public && cd public
##### vi script.js
##### vi index.html
<img width="389" alt="public script index" src="https://user-images.githubusercontent.com/112771723/191039784-4a469367-c156-43db-8f69-74ed2c1d6f1c.png"










