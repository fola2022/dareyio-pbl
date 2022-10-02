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






