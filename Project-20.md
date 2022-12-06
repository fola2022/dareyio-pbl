## MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER & DOCKER COMPOSE
Untill now we have using Ec2 instance to install and deploy our softwares. While this is easy and fast, it has its own challenges. Consider that you have the requirement into two set of softwares with both needing different version of a dependency say java. This is will lead to conflict. In software speaks it is called dependency matrix.

Another problem encountered in software development is the problem of IT WORKS IN MY COMPUTER . This problem arises from the configuration drift between the developers computer and the testers computer.

All the problem highlighted above is solved by containerization. Container solves this problem by creating an isolated environmentment using Linux features like NAMESPACE and CGROUP for a process.

containers are used to package application code, app configuration, dependencies and runtime environment required for running an application. This garanties to a large extent that the application runs efficiently and predictably on any environment it is deployed provider it has a container runtime.
### MySQL in container
### Step 1: Pulling MySQL Docker Image from Docker Hub Registry
```
docker pull mysql/mysql-server:latest
```
#### Listing the images to check that it downloaded successfully:
```
docker image ls
```
<img width="505" alt="mysql image" src="https://user-images.githubusercontent.com/112771723/205931366-5178b240-dc18-4bee-abca-36d0de6bd278.png">

#### Step 2: Deploying MySQL Container to my Docker Engine
```
docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest
```
#### Checking to see if the MySQL container is running
```
docker ps -a
```
<img width="938" alt="mysql running" src="https://user-images.githubusercontent.com/112771723/205934314-5a8178bc-8a74-4a22-bfd0-c36f14403881.png">

### CONNECTING TO THE MYSQL DOCKER CONTAINER
#### First, create a network:
```
docker network create --subnet=172.18.0.0/24 tooling_app_network
```
#### Create an environment variable to store the root password:
```
export MYSQL_PW=
```
#### Then, pulling the image and running the container, all in one command like below:
```
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW -d mysql/mysql-server:latest
```
<img width="746" alt="network" src="https://user-images.githubusercontent.com/112771723/205935248-2759e2ff-5825-4d3e-8a50-b9fbb3322b6b.png">

#### Because it's not a good practice to connect to MySQL server remotely using the root user. An sql file will be created for mysql user.
#### Creating a file create_user.sql and adding the following code in order to create a user: 
```
CREATE USER 'shade'@'%' IDENTIFIED BY 'password'; 
GRANT ALL PRIVILEGES ON *.* TO 'shade'@'%';
FLUSH PRIVILEGES;
```
#### Running the script to create the new user: 
```
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql
```

#### Connecting to the MySQL server from a second container running the MySQL client utility
Run the MySQL Client Container:

docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u -p

Prepare database schema
Clone the Tooling-app repository from [here](clone https://github.com/darey-devops/tooling.git)

git clone https://github.com/darey-devops/tooling.git

On your terminal, export the location of the SQL file

export tooling_db_schema=tooling/html/tooling_db_schema.sql

Verify that the path is exported:

echo $tooling_db_schema

Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container:

docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema

Update the .env file with connection details to the database:

             ```
                  sudo vi .env

               MYSQL_IP=mysqlserverhost
               MYSQL_USER=username
               MYSQL_PASS=client-secrete-password
               MYSQL_DBNAME=toolingdb

            ```
Run the Tooling App

Build the image using the Dockerfile in the clonned Repo

docker build -t tooling:0.0.1 .

Run the app with the command below:

docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1

You should find something like the screenshot below on your browser:


