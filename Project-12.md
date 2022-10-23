# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)
### This is a continuation of Project 11 Ansible-Config-mgt repository
### [This is the link to project 12 repository](https://github.com/fola2022/Ansible-Config-mgt.git)
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
<img width="401" alt="refactor branch created" src="https://user-images.githubusercontent.com/112771723/197399444-f4fe6c80-2b58-4c7d-ac7f-9927711fe694.png">

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

### CONFIGURing UAT WEBSERVERS WITH A ROLE ‘WEBSERVER'
#### Two Ec2 instances was used as webservers, named web1-uat and web2-uat. Also a roles directory was created on the jenkins/ansible instance inside the ansible-config-mgt repository and inside the roles folder, webserver folder was created.
```
sudo mkdir roles
```
<img width="425" alt="mkdir" src="https://user-images.githubusercontent.com/112771723/197399312-430e9de0-f9a2-462c-9383-27bd2ffaba03.png">

#### Updating inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of the 2 UAT Web servers
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```
<img width="328" alt="uat updated" src="https://user-images.githubusercontent.com/112771723/197399394-5bce3380-1d6b-4572-aa5e-be5b4fcb19e1.png">

#### In /etc/ansible/ansible.cfg file, the roles_path string was uncommented and the full path to the roles directory roles_path was provided so Ansible could know where to find configured roles.
<img width="497" alt="role config" src="https://user-images.githubusercontent.com/112771723/197399688-0ee850d0-3d0a-4655-9de4-e440c79cb84d.png">
<img width="377" alt="inventory config sudo vi etcansibleansiblecfg" src="https://user-images.githubusercontent.com/112771723/197399753-1d83bc66-a464-478a-91e7-6b361151d8f5.png">

#### The main.yml file in tasks directory was updated with a new task
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
### REFERENCE WEBSERVER ROLE
#### Within the static-assignments folder, uat-webservers.yml file was created and updated with the below code
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
#### Since the entry point of the configuration is site.yml, the file was updated with the below code
```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

#### Git save, commit and push was done to save the new update of the repository
```
git status
git add .
git commit -m 
git push origin refactor
```
<img width="514" alt="git add and status refac" src="https://user-images.githubusercontent.com/112771723/197400343-0cec34f0-3ee6-4f32-a56f-51f0cb8a4c79.png">
<img width="434" alt="git add new update" src="https://user-images.githubusercontent.com/112771723/197400362-8eb38fdf-7667-4a0d-9444-c0af9f8f842b.png">
<img width="561" alt="git push refac" src="https://user-images.githubusercontent.com/112771723/197400369-031aa96c-133e-456f-a1d7-4908dd987d40.png">

#### Running the playbook
```
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```
<img width="947" alt="playbook run last" src="https://user-images.githubusercontent.com/112771723/197400581-7ee6e26e-fb99-4112-a6b7-b707d0f925df.png">
<img width="651" alt="end" src="https://user-images.githubusercontent.com/112771723/197400632-93a12d1d-33db-4f50-934a-fa697f381374.png">




