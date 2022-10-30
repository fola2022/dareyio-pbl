# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
#### A new branch named dynamic-assignments was created in the Ansible-Config-mgt repository
[Here is the link to the repository](https://github.com/fola2022/Ansible-Config-mgt/tree/dynamic-assignments)
##### Dynamic-assignments directory was created and env-vars.yml was created in it. Site.yml would be instruct later to include this playbook.
##### Then env-vars folder was created to keep each environment variables file. The environment variable files dev.yml, stage.yml, uat.yml and prod.yml were created in the env-vars folders.
<img width="511" alt="dynamic and en var folder" src="https://user-images.githubusercontent.com/112771723/198902096-e9d06620-cd00-40b3-ba4b-c711c2da0ff8.png">

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
#### Mysql role was installed using the ansible galaxy role and configured
 <img width="433" alt="mysql installed" src="https://user-images.githubusercontent.com/112771723/198901528-0665dddc-55aa-4cf8-b962-22c9effd8140.png">
 <img width="482" alt="mysql config" src="https://user-images.githubusercontent.com/112771723/198901889-66597483-b1fe-475f-8e58-31377a6a77f2.png">
 
#### All process was uploaded to github
```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
<img width="533" alt="git add an commit dyna" src="https://user-images.githubusercontent.com/112771723/198901771-d289e341-2af7-427e-abd3-d57d98d1d650.png">
<img width="557" alt="git push dynamic" src="https://user-images.githubusercontent.com/112771723/198901787-1978de6f-84d6-4fe9-a934-a4085e6c9b1f.png">

### LOAD BALANCER ROLES
#### To choose which Load Balancer to use, Nginx or Apache,the two roles were installed respectively
<img width="455" alt="nginx role install" src="https://user-images.githubusercontent.com/112771723/198901658-2a239ac5-0add-4dd8-862b-f10c26126c1a.png">
<img width="461" alt="apache role" src="https://user-images.githubusercontent.com/112771723/198901703-c2cae35d-0a84-44a4-8497-e39f47e97e23.png">
<img width="532" alt="rename apache and nginx" src="https://user-images.githubusercontent.com/112771723/198901726-133b2ab9-029b-4cd1-92f7-c93d6dc65a76.png">

#### The loadbalancers.yml file in static-assignments folder was updated with the below code
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```
<img width="461" alt="lb yml updated use" src="https://user-images.githubusercontent.com/112771723/198901827-0042ce24-4ee6-4614-bff3-1c0a4c61b04f.png">

#### The site.yml file in playbooks folder was also updated with the below code
```
 - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
```
<img width="452" alt="latest site yml" src="https://user-images.githubusercontent.com/112771723/198901630-7e5d3373-31ce-4bf0-b47a-ad1aec92f8ea.png">

#### The uat.yml in env-vars folder was used to define which loadbalancer is to be enable
```
enable_apache_lb: true
load_balancer_is_required: true
```
<img width="469" alt="lb yml file updated" src="https://user-images.githubusercontent.com/112771723/198901582-ea3918f9-ea0d-478b-ada8-65d07d48ec18.png">

#### Running ansible
```
ansible-playbooks -i inventory/uat.yml playbooks/site.yml
```
<img width="942" alt="end" src="https://user-images.githubusercontent.com/112771723/198901996-e0784d9e-3778-40e8-970a-14b7018fa3f8.png">
<img width="688" alt="end2" src="https://user-images.githubusercontent.com/112771723/198902010-1fe01368-af01-4bef-9b7a-c4e7a335c379.png">

#### Checked apache to see apache running on the loadbalancer instance
<img width="690" alt="apache" src="https://user-images.githubusercontent.com/112771723/198902068-f19e3011-10e0-4e40-8344-bdcbfcb5b83e.png">


