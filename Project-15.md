## AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
### This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a company, who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this. The infrastructure will look like following diagram:
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project15/tooling_project_15.png)

### STEP 0: Setting Up a Sub-account
#### Creating a sub-account 'DevOps' from my AWS master account in the AWS Organisation Unit console
![Screenshot (467)](https://user-images.githubusercontent.com/112771723/202526536-28d5129e-4008-4ec4-b09b-0be3b5d77c69.png)

### STEP 1: SET UP A VIRTUAL PRIVATE NETWORK (VPC)
#### Created a VPC from the VPC console in AWS
#### Created subnets as shown in the above diagram 
<img width="693" alt="subnet" src="https://user-images.githubusercontent.com/112771723/202529406-786f4cf9-e60c-4e1c-be10-4c0cd42bb221.png">

#### Created a route table and associated it with public and private subnets respectively
<img width="695" alt="route table" src="https://user-images.githubusercontent.com/112771723/202529833-fb294993-ee22-448e-8be9-2ce2ed7ae087.png">

#### Created an Internet Gateway and it was attched to the VPC
![internet gateway](https://user-images.githubusercontent.com/112771723/202530715-4b6baaee-b2bb-4472-b344-3a3ffd3aca56.png)

#### Edited a route in public route table, and associated it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet).
#### Created 3 Elastic IPs
![elastic ip](https://user-images.githubusercontent.com/112771723/202531891-7e164c49-411e-4140-b068-45b3cb3d9040.png)

#### Created a Nat-gateway and one of the elastic ip address was assigned to it
![nat-gateway](https://user-images.githubusercontent.com/112771723/202532862-abe6ff7c-aa7e-4973-8e16-586b695c55e3.png)

#### Created Application load balancer (internal and external ALB)
<img width="742" alt="load balancer" src="https://user-images.githubusercontent.com/112771723/202536168-427d7c43-cbc9-49ab-a40d-2a6b726335ca.png">

### Created security group for:
#### - Application Load Balancer (ALB): ALB will be available from the Internet (HTTPS - Port 443 and HTTP - Port 80)
#### - Nginx server: Access only from the Application load balancer security group
<img width="818" alt="nginx reverse proxy security group" src="https://user-images.githubusercontent.com/112771723/202536349-ccf5edb4-6985-4068-aa64-d60a4c9ed2ba.png">

#### - Bastion server: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers.
#### - Webserver
<img width="809" alt="webserver security group" src="https://user-images.githubusercontent.com/112771723/202535801-f071a212-e692-47f6-866f-feb6f5578876.png">

#### - Datalayer: Allow TCP port 3306 traffic only from the webservers.
<img width="679" alt="All security group" src="https://user-images.githubusercontent.com/112771723/202533171-737045c9-155c-4e59-a52a-599596f827d2.png">

### STEP 2: Creating An AMI Out Of The EC2 Instance For Nginx And Bastion server
#### For Bastion Server 
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm 
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
yum install wget vim python3 telnet htop git mysql net-tools chrony -y 
systemctl start chronyd 
systemctl enable chronyd
```
<img width="487" alt="bastion ami" src="https://user-images.githubusercontent.com/112771723/202537986-627fa5ba-287d-4fa9-80a3-dda61dca1e02.png">

#### For Nginx server
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd
```
<img width="487" alt="nginx chroyd running" src="https://user-images.githubusercontent.com/112771723/202538545-c28dcdea-8bb8-4df4-92fa-f05aada83317.png">

#### For webserver
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd
```
<img width="737" alt="AMI" src="https://user-images.githubusercontent.com/112771723/202536551-733acdd0-4a4f-4225-9ee8-14fd795589fa.png">

#### Configured selinux policies for the webservers and nginx servers
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
### STEP 3: Creating A Launch Template
#### AMI is used to set up the launch template
<img width="693" alt="launch template" src="https://user-images.githubusercontent.com/112771723/202540903-41ddb852-f615-4303-b53a-b9f78a240caa.png">

#### Configured Userdata to update yum package repository and installed nginx
```
#!/bin/bash
sudo su -
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/fola2022/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config
```
### Configure Target Groups for Nginx, wordpress and tooling (webserver)
<img width="691" alt="target group" src="https://user-images.githubusercontent.com/112771723/202542803-1c151eaa-d14a-474c-b47a-fad6cb40db80.png">



















