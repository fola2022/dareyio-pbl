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
<img width="505" alt="mysql image" src="https://user-images.githubusercontent.com/112771723/205931366-5178b240-dc18-4bee-abca-36d0de6bd278.png">

#### Listing the images to check that it downloaded successfully:
```
docker image ls
```

Step 2: Deploy the MySQL Container to your Docker Engine

docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest
Then, check to see if the MySQL container is running: Assuming the container name specified is mysql-server

docker ps -a

CONNECTING TO THE MYSQL DO
