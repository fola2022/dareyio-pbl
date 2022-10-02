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
#### Mounting /var/www/html on apps-lv logical volume
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
#### Updating the /etc/fstab file for the mount configuration to persist after restart of the server
##### Commands:
```
sudo blkid
sudo vi /etc/fstab


<img width="405" alt="mount successfully on fstab" src="https://user-images.githubusercontent.com/112771723/193461169-4319a9d5-3fa4-4a03-bf81-4a0eb929280e.png">



































