## AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 4 – TERRAFORM CLOUD
### INTRODUCTION
#### In this project, instead of running the Terraform codes in project 18 from a command line, rather it is being executed from Terraform cloud console. The AMI is built differently with packer while Ansible is used to configure the infrastructure after its been provisioned by Terraform. 
### STEP 1: Setting Up A Terraform Account
#### After creating and account on the terraform console, an organisation was created in the terraform cloud site
#### To configure the workspace, version control workflow was selected in order to run terraform commands triggered from git repository
#### Creating a new repository called terraform-cloud and pushing the terraform codes developed in project 18 into the repository
#### Connecting the workspace to the new repository created and clicking on create workspace
<img width="911" alt="tf" src="https://user-images.githubusercontent.com/112771723/205124342-6c6ad8fc-3285-406d-8399-1b0b5b3ad18b.png">

#### Configuring the environment variable using my AWS credentials 
<img width="911" alt="env" src="https://user-images.githubusercontent.com/112771723/205124959-ed6459a9-d310-4525-ac89-e233dd48573c.png">

### STEP 2: Building AMI With Packer
#### Installing packer on my local machine
```
sudo apt install packer
```
#### - Cloning the repository and changing directory to the AMI folder
#### - Running the packer commands to build AMI for Bastion server, Nginx server and webserver
```
packer build bastion.pkr.hcl
packer build nginx.pkr.hcl
packer build web.pkr.hcl
packer build ubuntu.pkr.hcl
```
![Screenshot (604)](https://user-images.githubusercontent.com/112771723/205126291-de4f6236-caa4-4bf4-b818-aa102fedf16c.png)

### STEP 3: Running The Terraform Cloud To Provision Resources
#### - Inputing the AMI ID in my terraform.tfvars file for the servers built with packer which terraform will use to provision Bastion, Nginx, Tooling and Wordpress server
<img width="410" alt="update ami" src="https://user-images.githubusercontent.com/112771723/205129781-bae859e2-353d-47e5-8ba5-1861b4cbe8bd.png">

#### - Pushing the codes to my repository to trigger terraform plan on the terraform cloud 
#### - Accepting the plan to trigger an apply command
<img width="674" alt="tff" src="https://user-images.githubusercontent.com/112771723/205131147-d45af888-8889-436b-adfd-4e13b1bc4e82.png">
<img width="660" alt="tf2" src="https://user-images.githubusercontent.com/112771723/205131182-bfa1c58a-2672-4acb-bee6-80c75c45dbd2.png">
<img width="641" alt="tf3" src="https://user-images.githubusercontent.com/112771723/205131218-7c63a00c-d0f5-44fe-b271-243c36899781.png">

### STEP 4: Configuring The Infrastructure With Ansible
#### After a successful execution of terraform apply, connecting to the bastion server through ssh-agent to run ansible against the infrastructure
#### - Updating the nginx.conf.j2 file to input the internal load balancer DNS name generated via terraform:
<img width="686" alt="ng" src="https://user-images.githubusercontent.com/112771723/205137125-200db449-da7c-49a9-a353-e794646935b6.png">

#### - Updating the RDS endpoints, Database name, password and username in the setup-db.yml file for both the tooling and wordpress role
#### For Tooling 
<img width="677" alt="db tooling" src="https://user-images.githubusercontent.com/112771723/205137833-f0f58d75-7bba-4b47-8b26-529179e02696.png">

#### For Wordpress
<img width="683" alt="db wordpress" src="https://user-images.githubusercontent.com/112771723/205138578-d59e37d3-0640-47bf-870a-4eece3cedfbd.png">

#### - Updating EFS Access point ID for tooling and wordpress in the main.yml file of the tasks folder
##### EFS
<img width="905" alt="access point" src="https://user-images.githubusercontent.com/112771723/205138821-08adfc72-68d2-48ea-bd1c-947c831758f9.png">

#### For Tooling
<img width="593" alt="tooling efs" src="https://user-images.githubusercontent.com/112771723/205139511-7bd9eb49-b06c-46a6-9412-4ef883ebc8c7.png">

#### For Wordpress
<img width="650" alt="wordpress efs" src="https://user-images.githubusercontent.com/112771723/205139592-de980dc5-f327-4403-bb9b-e39057b7811b.png">

#### Exporting the environment variable ANSIBLE_CONFIG to point to the ansible.cfg from the repo and running the ansible-playbook command
#### Installing boto3
```
sudo python3.9 -m pip install boto3 botocore
```
![Screenshot (624)](https://user-images.githubusercontent.com/112771723/205140873-44bc877e-2947-4e4d-81fa-532b9a7a7774.png)

#### Running ansible command
```
ansible-playbook -i inventory/aws_ec2.yml playbook/site.yml
```
<img width="751" alt="ans1" src="https://user-images.githubusercontent.com/112771723/205141244-5b7df024-f893-4f01-8c6b-adc2197566d6.png">




#### Website end point 
```
dev-test-fola20221203141952295600000001.s3-website-us-east-1.amazonaws.com
```

