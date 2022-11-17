## AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
### This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a company, who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this. The infrastructure will look like following diagram:
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project15/tooling_project_15.png)

### STEP 0: Setting Up a Sub-account
#### Creating a sub-account 'DevOps' from my AWS master account in the AWS Organisation Unit console
![Screenshot (467)](https://user-images.githubusercontent.com/112771723/202526536-28d5129e-4008-4ec4-b09b-0be3b5d77c69.png)

### STEP 1: Creating a hosted zone in the Route 53 console and mapping it to the domain name acquired from freenom
![Screenshot (491)](https://user-images.githubusercontent.com/112771723/202556734-7b24eec4-a949-4e39-959b-9704f59b1606.png)

### STEP 2: SET UP A VIRTUAL PRIVATE NETWORK (VPC)
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

### Created security group for:
#### - Application Load Balancer (ALB) security group: ALB will be available from the Internet (HTTPS - Port 443 and HTTP - Port 80)
#### - Nginx server: Access only from the Application load balancer security group
<img width="818" alt="nginx reverse proxy security group" src="https://user-images.githubusercontent.com/112771723/202536349-ccf5edb4-6985-4068-aa64-d60a4c9ed2ba.png">

#### - Bastion server: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers.
#### - Webserver
<img width="809" alt="webserver security group" src="https://user-images.githubusercontent.com/112771723/202535801-f071a212-e692-47f6-866f-feb6f5578876.png">

#### - Datalayer: Allow TCP port 3306 traffic only from the webservers.
<img width="679" alt="All security group" src="https://user-images.githubusercontent.com/112771723/202533171-737045c9-155c-4e59-a52a-599596f827d2.png">

### STEP 3: Creating An AMI Out Of The EC2 Instance For Nginx And Bastion server
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
### STEP 4: Creating A Launch Template
#### AMI is used to set up the launch template
### For Bastion Server
#### - Setting up a launch template with the Bastion AMI
#### - Ensuring the Instances are launched into the public subnet
#### - Entering the Userdata to update yum package repository and install ansible and mysql
```
#!/bin/bash
yum install -y mysql
yum install -y git tmux
yum install -y ansible
```
### For Nginx Server
#### - Setting up a launch template with the Nginx AMI
#### - Ensuring the Instances are launched into the public subnet
#### - Assigning appropriate security group
#### - Entering the Userdata to update yum package repository and install Nginx

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
### For Tooling Server
#### - Setting up a launch template with the Bastion AMI
#### - Ensuring the Instances are launched into the public subnet
#### - Assigning appropriate security group
#### - Entering the Userdata to update yum package repository and apache server
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-053e953bddd2eff1b fs-08e7b5b3eba1146b4:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/fola2022/tooling-1.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h acs-rds-database.cv3syqptmlls.us-east-1.rds.amazonaws.com -u ACSadmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('acs-rds-database.cv3syqptmlls.us-east-1.rds.amazonaws.com', 'ACSadmin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```
### For Wordpress server
#### - Setting up a launch template with the Bastion AMI
#### - Ensuring the Instances are launched into the public subnet
#### - Assigning appropriate security group
#### - Configure Userdata to update yum package repository and install wordpress and apache server
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls fs-08e7b5b3eba1146b4:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/acs-rds-database.cv3syqptmlls.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```
<img width="693" alt="launch template" src="https://user-images.githubusercontent.com/112771723/202540903-41ddb852-f615-4303-b53a-b9f78a240caa.png">

### STEP 5: Configure Target Groups for Nginx, wordpress and tooling 
#### Selecting Instances as the target type
#### Ensuring the protocol HTTPS on secure TLS port 443
#### Ensuring that the health check path is /healthstatus
<img width="691" alt="target group" src="https://user-images.githubusercontent.com/112771723/202542803-1c151eaa-d14a-474c-b47a-fad6cb40db80.png">

###  Configuring AutoScaling Group for Ngnix, Bastion, Tooling and Wordpress server
#### - Selecting the right launch template
#### - Selecting the VPC
#### - Selecting both public subnets
#### - Enabling Application Load Balancer for the AutoScalingGroup (ASG)
#### - Selecting the target group created before
#### - Ensuring health checks for both EC2 and ALB
#### - Setting the desired capacity, Minimum capacity and Maximum capacity to 2
#### - Setting the scale out option if CPU utilization reaches 90%
#### - Activating SNS topic to send scaling notifications

### STEP 6: TLS Certificates From Amazon Certificate Manager (ACM)
#### TLS certificates is created to handle secured connectivity to the Application Load Balancers (ALB)
#### Using AWS ACM and Requested for a public certificate for the domain name registered on Freenom
#### Using DNS to validate the domain name
### Configuring Application Load Balancer (ALB)
### For External Load Balancer
#### - Selecting Internet facing option
#### - Ensuring that it listens on HTTPS protocol (TCP port 443)
#### - Ensuring the ALB is created within the appropriate VPC, AZ and the right Subnets
#### - Choosing the Certificate already created from ACM
#### - Selecting Security Group for the external load balancer
#### - Selecting Nginx Instances as the target group
### For Internal Load Balancer
#### - Selecting Internet facing option
#### - Ensuring that it listens on HTTPS protocol (TCP port 443)
#### - Ensuring the ALB is created within the appropriate VPC, AZ and the right Subnets
#### - Choosing the Certificate already created from ACM
#### - Selecting Security Group for the internal load balancer
#### - Selecting webserver Instances as the target group
#### - Ensuring that health check passes for the target group
<img width="742" alt="load balancer" src="https://user-images.githubusercontent.com/112771723/202559773-6e4b32ce-a765-41b0-8ed6-aaeefbaeb56a.png">

### STEP 7: Setting Up EFS Storage For The Webservers
#### - Created an EFS filesystem and mount target per AZ in the VPC, which was associated with both subnets dedicated for data layer. The security group created for data layer was associated to it. Also, an EFS access point was created for both wordpress and tooling server.

<img width="666" alt="file system created" src="https://user-images.githubusercontent.com/112771723/202560875-5076723a-9a7e-4c1b-892a-4fa1a57121ee.png">
<img width="652" alt="efs access point" src="https://user-images.githubusercontent.com/112771723/202560933-b27083b5-0691-46d8-998a-25a64101ea1d.png">

### STEP 8: SETTING UP RDS
#### A Key Management Service(KMS) was created in AWS, which was used to encrypt the database instance.
<img width="705" alt="kms" src="https://user-images.githubusercontent.com/112771723/202562260-97f19692-769e-4263-bb8d-b61805c83511.png">

#### Setting Up A Relational Database System
<img width="704" alt="database" src="https://user-images.githubusercontent.com/112771723/202563095-73899713-e55d-42e3-87e8-7ddb6f544383.png">

####  DNS Records were created in The Route53 For the Tooling And Wordporess site
#### The url https://www.tooling.yellowgem.tk was access on the broswer. The url route traffic from the application load balancer to the nginx and then to the internal ALB which forwards the traffic to the server for tooling site:
[Link to the repository](https://github.com/fola2022/ACS-project-config)

<img width="473" alt="end" src="https://user-images.githubusercontent.com/112771723/202563768-3d69d087-63d1-4e46-9d0c-8616dfb15927.png">











