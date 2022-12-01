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
