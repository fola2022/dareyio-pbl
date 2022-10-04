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























