## DEVOPS TOOLING WEBSITE SOLUTION
#### Four EC2 instance were used
##### One for NFS Server
##### Three webservers
##### One database server
### STEP 1: PREPARING NFS SERVER
#### Three volume, each of 10GB were created and attached to the NFS server EC2 instance
##### Commands:
``` 
lsblk
df -h
```  
<img width="284" alt="1" src="https://user-images.githubusercontent.com/112771723/193802327-506bbe90-5e5b-428a-8c63-2f6f941d31eb.png">

####  Creating a single partition on each of the 3 disks
##### Commands:
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
lsblk
```
<img width="502" alt="part 1" src="https://user-images.githubusercontent.com/112771723/193802521-38cc5462-3f9f-406b-b1d3-869e423d57a0.png">
<img width="513" alt="part 2" src="https://user-images.githubusercontent.com/112771723/193802605-53f08f9b-5736-4acd-9e73-86ac68b3b726.png">
<img width="502" alt="part 3" src="https://user-images.githubusercontent.com/112771723/193802622-9a1b2b29-4656-4614-988b-9d3e4036ed79.png">
<img width="285" alt="part  config" src="https://user-images.githubusercontent.com/112771723/193802692-cc0efbd2-3d5c-4110-8f1f-225c2144b741.png">

#### Installing lvm2 package
##### Commands:
```
sudo yum install lvm2 -y
sudo lvmdiskscan 
```
<img width="522" alt="lvm2 installed" src="https://user-images.githubusercontent.com/112771723/193802849-318000d8-c459-4a99-880b-7c45bbf2b6cd.png">
<img width="346" alt="lvmdiskscan" src="https://user-images.githubusercontent.com/112771723/193802874-9683386c-f157-4c53-9d14-3151b0d68614.png">

####  The 3 disks were marked as physical volumes (PVs) to be used by LVM, then volume group was also created
##### Commands:
```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
sudo pvs
sudo vgs
```
<img width="519" alt="physical volume and volume group ceated" src="https://user-images.githubusercontent.com/112771723/193803137-7f60f9eb-f39a-41de-af5b-3312bcd66857.png">

#### Creating 3 logical volumes. lv-apps, lv-logs and lv-opt
##### Commands:
```
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
sudo lvs
```
<img width="529" alt="logical volume" src="https://user-images.githubusercontent.com/112771723/193804879-dd6ee5d3-23ee-4ba6-96c0-c7bfbfebfd9e.png">

#### Using mkfs.ext4 to format the logical volumes with ext4 filesystem
##### Commands:
```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```
<img width="499" alt="logical volume formartted mkfs -t" src="https://user-images.githubusercontent.com/112771723/193805383-0e6b249e-4ec1-4450-b0d4-e58b5f6e1645.png">

#### Creating /mnt directory to mount logical volumes
##### Commands:
```
sudo mkdir /mnt/apps
sudo mkdir /mnt/logs
sudo mkdir /mnt/opt
```
<img width="317" alt="mnt directory" src="https://user-images.githubusercontent.com/112771723/193806340-52e265d3-b8de-4889-945d-f56240057ad8.png">

#### Mounting lv-apps, lv-logs and lv-opt logical volumes on /mnt directories  
```
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
#### Updating the /etc/fstab file for the mount configuration to persist after restart of the server
##### Commands:
```
sudo blkid
sudo vi /etc/fstab
sudo mount -a
sudo systemctl daemon reload
```
<img width="512" alt="block id" src="https://user-images.githubusercontent.com/112771723/193809472-e0504835-dd90-45e0-9111-175e390c736d.png">
<img width="654" alt="fstab" src="https://user-images.githubusercontent.com/112771723/193809534-dab1b010-ee4c-42ce-bdf5-32c13faa2b26.png">
<img width="478" alt="successfully mounted fstab" src="https://user-images.githubusercontent.com/112771723/193809565-8f9bb42b-ff3e-4b29-b337-fe32a0c6fde0.png">

### Installing NFS server
##### Commands:
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
<img width="514" alt="nfs conf 1 yum update" src="https://user-images.githubusercontent.com/112771723/193810597-0e57bb22-0762-48fc-9cfb-9a90ddb33663.png">
<img width="524" alt="nfs conf  2" src="https://user-images.githubusercontent.com/112771723/193810670-682093af-5b35-411b-8d19-5cf666b570ea.png">
<img width="514" alt="nfs conf  345" src="https://user-images.githubusercontent.com/112771723/193810704-3a1d61bf-7a63-4e63-91e4-1559ca06ac88.png">

#### Webservers subnet cidr was exported to connect as clients. That's three Web Servers would be install inside the same subnet
#### Permission that would allow the three Web servers to read, write and execute files on NFS were set.
##### Commands:
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
sudo systemctl restart nfs-server.service
```
<img width="490" alt="set pernission nfs" src="https://user-images.githubusercontent.com/112771723/193812952-e2b0da34-a32c-4c63-b4d6-277d95f10f9c.png">

#### Configuring access to NFS for clients within the same subnet 
##### Commands:
```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
sudo exportfs -arv
```
<img width="504" alt="config subnet in vi etc nfs" src="https://user-images.githubusercontent.com/112771723/193813555-0ad9f2ec-c244-4509-9b92-c1d247d40c0e.png">
<img width="475" alt="confi  access nfs" src="https://user-images.githubusercontent.com/112771723/193813585-822ffba4-d041-48ff-9569-eb65dbdc9031.png">

#### Checking NFS Port also in order for NFS server to be accessible from client server, the following ports were opened TCP 111, UDP 111, UDP 2049
##### Command `rpcinfo -p | grep nfs`
<img width="337" alt="nfs port" src="https://user-images.githubusercontent.com/112771723/193813860-2b7b5e04-b147-4f9c-9af8-c7210d2c5dee.png">

## STEP 2:  CONFIGURING THE DATABASE SERVER
#### Installing mysql server and creating a database
##### Command: `sudo apt install mysql-server -y`
<img width="510" alt="mysql installed on db" src="https://user-images.githubusercontent.com/112771723/193814597-9d8cfbd2-f96b-4bc0-a043-59c6e616955c.png">
<img width="512" alt="config  mysql" src="https://user-images.githubusercontent.com/112771723/193815274-7421a1d9-6394-4430-8f6b-e44e9cd6bcb6.png">

## STEP 3:  Preparing the Web Servers
#### Installing NFS client
##### Command: `sudo yum install nfs-utils nfs4-acl-tools -y`
<img width="515" alt="nfs client installed on webserver1" src="https://user-images.githubusercontent.com/112771723/193816640-abd8823b-692d-443b-86f8-eade0d6c943c.png">

#### Mount /var/www and target the NFS server’s export for apps
##### Commands:
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
sudo vi /etc/fstab
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```
<img width="523" alt="nfs mounted succ on webse1 use" src="https://user-images.githubusercontent.com/112771723/193816718-437d7404-bd76-42af-ae86-1f3848b6ce15.png">

#### Installing Remi’s repository, Apache and PHP
##### Commands:
```
sudo yum install httpd -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
<img width="520" alt="apache installed webser1 1" src="https://user-images.githubusercontent.com/112771723/193817575-9151b5c4-5e8c-4e30-aa95-a3a8b18eacb3.png">
<img width="523" alt="2" src="https://user-images.githubusercontent.com/112771723/193817633-2ab9a650-392a-437a-ae29-d0f0437eb532.png">
<img width="514" alt="apache3" src="https://user-images.githubusercontent.com/112771723/193817653-220fdfe2-63a0-4d12-be9d-6ec25e17791b.png">
<img width="512" alt="apache 4" src="https://user-images.githubusercontent.com/112771723/193817669-ce558c75-2c31-4306-8b7d-abba18e4781b.png">
<img width="515" alt="apache 5" src="https://user-images.githubusercontent.com/112771723/193817723-f84ce3c8-f1dc-4ae2-9b30-f585b26bc42d.png">
<img width="514" alt="apache 6<img width="515" alt="apache remaing" src="https://user-images.githubusercontent.com/112771723/193818038-5867004c-4c91-4373-a132-69830dffff79.png">
<img width="515" alt="apache remaing" src="https://user-images.githubusercontent.com/112771723/193818139-ec1d00e3-79bb-44fc-97c4-48d1788f78e9.png">
                                                                                                         
                                                                                                                                              

















