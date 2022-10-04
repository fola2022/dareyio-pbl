## WEB SOLUTION WITH WORDPRESS
### STEP 1: Prepare a Web Server
#### Three volume, each of 10GB were created and attached to the web server EC2 instance
##### Commands:
``` 
lsblk
df -h
```  
<img width="367" alt="volume attached lsblk df" src="https://user-images.githubusercontent.com/112771723/193455497-92229c8b-54b6-4dfd-bee2-b89917cb61eb.png">

####  Creating a single partition on each of the 3 disks
##### Commands:
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
lsblk
```
<img width="517" alt="partition1" src="https://user-images.githubusercontent.com/112771723/193455916-3ad42478-7e4e-4278-ada3-13fe0cb09aa5.png">
<img width="503" alt="partition2" src="https://user-images.githubusercontent.com/112771723/193455925-52817850-cf00-4a3a-a469-d09cc566aa30.png">
<img width="490" alt="partition3" src="https://user-images.githubusercontent.com/112771723/193455929-d0cf214d-761f-43c9-832a-4de90223613f.png">
<img width="328" alt="configure parti" src="https://user-images.githubusercontent.com/112771723/193456055-056cb940-f2c3-4d07-adcc-2122fa53a036.png">

#### Installing lvm2 package
##### Commands:
```
sudo yum install lvm2 -y
sudo lvmdiskscan 
```
<img width="509" alt="lvm2 installed" src="https://user-images.githubusercontent.com/112771723/193456445-c07e7f81-d2db-4008-8321-6f8c0d4e5e81.png">
<img width="308" alt="sudo lvmdiskscan" src="https://user-images.githubusercontent.com/112771723/193456402-a30c3251-4f2d-4a0c-9ba1-d8a4ab38937c.png">

####  The 3 disks were marked as physical volumes (PVs) to be used by LVM, then volume group was also created
##### Commands:
```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
sudo pvs
sudo vgs
```
<img width="492" alt="pvcreate" src="https://user-images.githubusercontent.com/112771723/193456739-6cb5d17e-a888-4ecd-ab80-50f9da2d1bf0.png">
<img width="515" alt="volume group created" src="https://user-images.githubusercontent.com/112771723/193456781-af4c26fb-4a45-4424-ae27-6ab2d9815a40.png">

#### Creating 2 logical volumes. apps-lv and logs-lv 
##### Commands:
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
sudo lvs
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
<img width="557" alt="logical volume created" src="https://user-images.githubusercontent.com/112771723/193459618-57c80282-7585-4ef8-8a46-7cbd3bc329c5.png">
<img width="514" alt="complete setting 1" src="https://user-images.githubusercontent.com/112771723/193459742-485dcac6-934b-4610-9d77-f46b14e6e05f.png">
<img width="504" alt="complete setting2" src="https://user-images.githubusercontent.com/112771723/193459763-80e8c9b3-4eb0-4a8e-b2ab-2a80f123a7e1.png">
<img width="508" alt="complete setting3" src="https://user-images.githubusercontent.com/112771723/193459770-cefdf510-d12f-4a02-b79a-bd40dd16be9a.png">
<img width="440" alt="complete setting 4" src="https://user-images.githubusercontent.com/112771723/193459777-4a199d60-38d6-4c8c-9236-8adcf4668ea7.png">

#### Using mkfs.ext4 to format the logical volumes with ext4 filesystem
##### Commands:
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
<img width="497" alt="logical voume formated" src="https://user-images.githubusercontent.com/112771723/193459916-55ea9636-a30b-45de-85b6-b3fc75225ac5.png">

#### Creating /var/www/html directory and /home/recovery/logs directory for backup of log data
##### Commands:
```
sudo mkdir -p /home/recovery/logs
sudo mkdir -p /home/recovery/logs
```
#### Backing up all the files in log directory /var/log
##### Command:
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
#### Mounting app-lv and log-lv logical volumes on /var/www/html and /var/log directory repectively  
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
sudo mount /dev/webdata-vg/logs-lv /var/log
```
<img width="466" alt="mounted" src="https://user-images.githubusercontent.com/112771723/193460772-5496f324-9bb8-48e3-bd28-8b998618551a.png">

#### Restoring all files in log directoty /var/log
#### Command:
```
sudo rsync -av /home/recovery/logs/. /var/log
```
<img width="488" alt="recover log folder" src="https://user-images.githubusercontent.com/112771723/193462775-16a38e26-4acb-41c7-a7f3-277ed3caead6.png">

#### Updating the /etc/fstab file for the mount configuration to persist after restart of the server
##### Commands:
```
sudo blkid
sudo vi /etc/fstab
sudo mount -a
sudo systemctl daemon reload
```
<img width="509" alt="fstab" src="https://user-images.githubusercontent.com/112771723/193461539-fa71f807-735d-4b4c-9e54-00c4fffa58e6.png">
<img width="405" alt="mount successfully on fstab" src="https://user-images.githubusercontent.com/112771723/193461667-a5434217-d727-4edc-8b01-4e5e665891df.png">
<img width="405"<img width="473" alt="after mounting" src="https://user-images.githubusercontent.com/112771723/193461572-545b1e13-c6a1-4f1f-a9aa-d5b99ed3390b.png">

### STEP 2: Prepare the Database Server
### All step 1 procedure were repeated for the Database Server, but here, only one logical volume db-lv would be created and mount to /db directory instead of /var/www/html/
#### Creating logical volumes db-lv 
##### Commands:
```
sudo lvcreate -n db-lv -L 20G database-vg
sudo lvs
sudo lsblk
```
<img width="519" alt="logical volume created db" src="https://user-images.githubusercontent.com/112771723/193462092-e876268c-a6d3-4ee6-844e-7916b833b28f.png">

#### Using mkfs.ext4 to format the logical volumes with ext4 filesystem
##### Command:
```
sudo mkdir /db
sudo mkfs -t ext4 /dev/database-vg/db-lv 
```
<img width="499" alt="logical volume formated db" src="https://user-images.githubusercontent.com/112771723/193462450-831a4a6e-a1f1-49ae-a41e-b6b1e1c2ebd0.png">

#### Mounting /var/www/html on apps-lv logical volume
```
sudo mount /dev/database-vg/db-lv /db
```
<img width="466" alt="logical v mounted db" src="https://user-images.githubusercontent.com/112771723/193462351-ea41bc1e-6d54-49f3-aa32-cc991cee9658.png">

#### Updating the /etc/fstab file for the mount configuration to persist after restart of the server
##### Commands:
```
sudo blkid
sudo vi /etc/fstab
sudo mount -a
sudo systemctl daemon reload
```
<img width="437" alt="fstab db" src="https://user-images.githubusercontent.com/112771723/193462708-a113a0ab-7dbc-4e89-b2b0-5e370e459087.png">

### STEP 3: Install WordPress on your Web Server EC2
#### Update repository and Install wget, Apache and it’s dependencies
##### Command:
```
sudo yum -y update
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
<img width="513" alt="wget" src="https://user-images.githubusercontent.com/112771723/193463128-70ad1a49-511b-4844-98d4-780919014836.png">

#### Starting Apache
##### Commands:
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
<img width="515" alt="httpd running" src="https://user-images.githubusercontent.com/112771723/193463226-344154cf-9f87-4edd-9199-f3b7c31dee6a.png">

####  Installing PHP and it’s dependencies
##### Commands:
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
<img width="511" alt="php installed" src="https://user-images.githubusercontent.com/112771723/193463452-24b074b2-3fcb-4406-98b1-91f267699f88.png">
<img width="515" alt="php 2" src="https://user-images.githubusercontent.com/112771723/193463460-5023b34c-a22e-4dc7-b5d0-3b444ff1b5df.png">
<img width="505" alt="php 3" src="https://user-images.githubusercontent.com/112771723/193463468-1aec05bd-72b0-4bfb-83c8-61b22fbd70cc.png">
<img width="514" alt="php 4" src="https://user-images.githubusercontent.com/112771723/193463502-7614a5b5-917f-42ec-ad06-cd90831f705a.png">
<img width="512" alt="php 5" src="https://user-images.githubusercontent.com/112771723/193463511-9df85223-0601-4686-8ec9-6fcb7194ad38.png">
<img width="505" alt="php 6" src="https://user-images.githubusercontent.com/112771723/193463519-e021db11-9ac2-46a7-996a-a6c9a2572deb.png">
<img width="517" alt="php running use" src="https://user-images.githubusercontent.com/112771723/193463582-a47429d5-8795-4683-8f91-a9baac951b7c.png">

#### Restarting Apache
##### Command
```
sudo systemctl restart httpd
```
#### Downloading wordpress and copy wordpress to var/www/html
##### Commands:
```
 mkdir wordpress
 cd wordpress
 sudo wget http://wordpress.org/latest.tar.gz
 sudo tar xzvf latest.tar.gz
 sudo rm -rf latest.tar.gz
 sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
 sudo cp -R wordpress /var/www/html/
 ```
 <img width="509" alt="http wordpress sudo wget" src="https://user-images.githubusercontent.com/112771723/193463943-bccc9031-0f73-4669-a1b6-3675ca1e8b7e.png">
<img width="370" alt="sudo tar x wordpress" src="https://user-images.githubusercontent.com/112771723/193463898-5df35a1c-f573-4570-8d96-47dbab37f40e.png">
<img width="512" alt="html directory" src="https://user-images.githubusercontent.com/112771723/193464466-c0c31bbc-8f54-4975-bf14-61af63463e6b.png">

#### Configuring SELinux Policies
##### Commands
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P htpd_can_network_connect_db 1
```
<img width="532" alt="selinux conf" src="https://user-images.githubusercontent.com/112771723/193464661-59465383-dbdb-4ef1-a2dd-7c7b6ee8c0cc.png">

### STEP 4: Installing MySQL on DB Server EC2
##### Commands:
```
sudo yum update
sudo yum install mysql-server
```
<img width="419" alt="install mysql" src="https://user-images.githubusercontent.com/112771723/193464788-432dcee3-54b0-467e-9e1a-beebc6f11bd6.png">

#### Verifying service is up and running
##### Commands:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
<img width="518" alt="mysql running in db" src="https://user-images.githubusercontent.com/112771723/193465146-b54791bb-46c8-4e33-a895-c1f02a79fe53.png">

### STEP 5: Configuring Database to work with WordPress
##### Commands:
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
<img width="509" alt="inside mysql db" src="https://user-images.githubusercontent.com/112771723/193465187-84231bc1-bb79-404f-9d92-6a290f473ab8.png">

### STEP 6: Configuring WordPress to connect to remote database
#### First, MySQL port 3306 was opened on DB Server EC2 and access was only allowed to the database server from the Web Server’s IP address
#### Installing Mysql client
##### Commands:
```
sudo yum install mysql-client
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
<img width="503" alt="webser data use at the end of the project" src="https://user-images.githubusercontent.com/112771723/193465559-fa4bbf4a-4f93-4dd9-8fb4-1255c6a9ac7a.png">
<img width="524" alt="mysql running in webserver" src="https://user-images.githubusercontent.com/112771723/193465500-e8510e22-feb1-4f66-a30a-0dc4bd7f0c74.png">
<img width="456" alt="showdatabase" src="https://user-images.githubusercontent.com/112771723/193465540-77e10ba8-cac3-4ce7-9045-07dd1ec1739c.png">

#### Changing permission and configuration so Apache could use WordPress
#### Command:
```
sudo vi wp-config.php
sudo systemctl restart hhtpd
```
<img width="517" alt="wp-conf webser use this" src="https://user-images.githubusercontent.com/112771723/193465892-26a4c318-974b-423f-a1ed-7f6915ab126d.png">

#### Enabling TCP port 80 in Inbound Rules configuration for Web Server EC2. It was enabled from everywhere 0.0.0.0/0 
<img width="629" alt="end" src="https://user-images.githubusercontent.com/112771723/193466012-763e6dba-3bc2-4f94-b329-176cb8396961.png">
<img width="944" alt="end 2" src="https://user-images.githubusercontent.com/112771723/193466016-83e5b67b-a257-4256-ba03-b233cb1da41c.png">





















































