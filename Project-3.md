## SIMPLE TO-DO APPLICATION ON MERN WEB STACK
### Step 1: Backend Configuration
#### Commands are; 
#####  sudo apt update
#####  sudo apt upgrade
#####  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
<img width="541" alt="sudo apt update and ungrade" src="https://user-images.githubusercontent.com/112771723/190863546-d3d38051-bd0f-4108-aeab-c54d1bbc12e7.png">
<img width="775" alt="curl" src="https://user-images.githubusercontent.com/112771723/190863564-e2e6776e-4e4a-49fa-a4aa-83ccf4255998.png">

### Installing Node.js and NPM pacakge
####  Command: sudo apt-get install -y nodejs
<img width="707" alt="install node js" src="https://user-images.githubusercontent.com/112771723/190863686-ec7e38c2-d040-4d6b-85f7-9356fdf9a35c.png">

### Application Code Setup and initialization of project
#### Commands are; 
##### mkdir Todo
##### npm init
<img width="427" alt="npm init" src="https://user-images.githubusercontent.com/112771723/190863990-59a699d8-1605-4af0-9de0-993cec6192fa.png">

### Installing Express Js and Installing dotenv module
#### A file name index.js was created and a code to allow port 5000 was placed in it.
#### Commands are;
##### npm install express
##### touch index.js
##### npm install dotenv
##### vim index.js
##### node index.js
<img width="348" alt="install express" src="https://user-images.githubusercontent.com/112771723/190864457-35be763b-a554-4898-b5d7-946796550476.png">
<img width="335" alt="installl doenv" src="https://user-images.githubusercontent.com/112771723/190864671-c93c7688-a1a4-4064-a72e-ad189f37a9b5.png">

### Accessing server on browser by add port 5000 to inbound rules in the EC2 instance
<img width="296" alt="port5000 on brower" src="https://user-images.githubusercontent.com/112771723/190865103-9534f228-f8f2-438a-89b8-dc634f3ade5f.png">

### Routes was created to define the various endpoints the To-do application would depend on.
#### Commands;
##### mkdir routes
##### cd routes
##### touch api.js
##### vim api.js
<img width="347" alt="routes dir" src="https://user-images.githubusercontent.com/112771723/190865599-932d02a9-bc13-4d4a-8b77-86117ed09fdd.png">
<img width="361" alt="routes api js" src="https://user-images.githubusercontent.com/112771723/190865631-37f2d992-d3c0-4e9e-9c7c-7186b1eb2dbf.png">

### MODELS: To create a schema and models, mongoose was installed and mongoose code was placed in the the todo.js file. The api.js file in the routes directory was update.
#### Commands are;
##### npm install mongoose
##### vim api.js
<img width="337" alt="install mongose model todo js" src="https://user-images.githubusercontent.com/112771723/190866014-9be5f1a7-c3d9-4e15-9009-b309567cea88.png">
<img width="361" alt="routes api js" src="https://user-images.githubusercontent.com/112771723/190866176-ec1cab7a-e694-4d21-9cd6-f8f564d564a5.png">

### MongoDB Database: Using mLab, a mongodb database and collection was created. The cluster was created to get connection string to access the database by creating a file called .env in the To-do directory. The code inside the file index.js was also update. The database was successfully connected.
#### Commands;
##### touch .env
##### vim .env
##### node index.js
<img width="659" alt="mongodb database in env file" src="https://user-images.githubusercontent.com/112771723/190866848-02e02fae-83c1-46e9-9cf6-d86f750c7bd6.png">
<img width="311" alt="databade connected successfully" src="https://user-images.githubusercontent.com/112771723/190866868-8a75109f-49d3-4661-980d-3a9e57378718.png">

### Testing Backend Code without Frontend using RESTful API
<img width="545" alt="post on postman" src="https://user-images.githubusercontent.com/112771723/190867028-4813147e-7f2a-4a90-a8b6-181d5b5c5297.png">
<img width="544" alt="GET postman1" src="https://user-images.githubusercontent.com/112771723/190867034-2ed945e0-6879-4eda-92e5-b16eece14e1c.png">










