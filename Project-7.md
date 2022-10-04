## DEVOPS TOOLING WEBSITE SOLUTION
#### Fours EC2 instance were used
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

#### Creating 3 logical volumes. apps-lv, logs-lv and opt-lv

