# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)
### This is a continuation of Project 11 Ansible-Config-mgt repository
### Link
### First, directory called ansible-config-artifact was created on the jenkin-ansible server to store all artifacts after each build in other to save space on the server and Jenkins was configured to copy artifact and a new freestyle project called save_artifact was created as the downstream project for ansible project.
##### Command: 
```
sudo mkdir /home/ubuntu/ansible-config-artifact
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
<img width="545" alt="make dir" src="https://user-images.githubusercontent.com/112771723/197396924-40035fe3-4aec-450f-a097-3509ec0b7fee.png">

### REFACTORING ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
#### In the Ansible-Config-mgt repository, a new branch was created, named refactor
#### In the playbooks directory, a site.yml file was created to serve as an entry point into th entire infrastructure configuration
#### Static-assignments folder was created in the Ansible-Config-mgt repository
#### Commom.yml file was moved to static-assignments folder
#### site.yml file import to commom.yml playbook
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
![Screenshot (305)](https://user-images.githubusercontent.com/112771723/197397999-ced60417-8fa2-45ae-9d38-d831318b8d31.png)
<img width="428" alt="site to import common" src="https://user-images.githubusercontent.com/112771723/197397787-ddaafc50-c1e7-462c-acab-392b64492aca.png">

#### common-del.yml playbook file created under static-assignments, would be use for configuration to delete wireshark
<img width="528" alt="common-del yml file" src="https://user-images.githubusercontent.com/112771723/197398453-6e4e2d69-402d-4a43-9662-fc64c073d74e.png">
<img width="531" alt="common to static" src="https://user-images.githubusercontent.com/112771723/197398489-a2f20341-d157-4e99-9aef-aa3446ce8ada.png">

#### The common-del.yml file
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```
#### Running ansible
```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```
<img width="802" alt="ansible run" src="https://user-images.githubusercontent.com/112771723/197398724-a20962c9-d372-41c0-8c27-eeea1b594b46.png">
<img width="409" alt="wireshark deleted on lb" src="https://user-images.githubusercontent.com/112771723/197398817-f9479983-29e3-49b9-8558-c08a73d86572.png">
<img width="380" alt="no wireshark nfs" src="https://user-images.githubusercontent.com/112771723/197398836-261cb929-2f23-46c4-973e-9dee86c1880a.png">


















