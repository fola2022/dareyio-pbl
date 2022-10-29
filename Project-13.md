## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
#### A new branch named dynamic-assignments was created in the Ansible-Config-mgt repository
[Here is the link to the repository](https://github.com/fola2022/Ansible-Config-mgt/tree/dynamic-assignments)
##### Dynamic-assignments directory was created and env-vars.yml was created in it. Site.yml would be instruct later to include this playbook.
##### Then env-vars folder was created to keep each environment variables file. The environment variable files dev.yml, stage.yml, uat.yml and prod.yml were created in the env-vars folders.
#### The below code was placed in the env-vars.yml file
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
  ```
  
  #### The site.yml file in playbooks directory was updated 
  ```
  ---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
  ```
  ##### Mysql role was installed using the ansible galaxy role
  #### 
  
