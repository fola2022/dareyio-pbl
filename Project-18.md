## AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3 – REFACTORING
### In continuation to [Project 17](https://github.com/fola2022/dareyio-pbl/blob/main/Project-17.md), the entire code is refactored in order to simplify the code using a Terraform tool called Module.
### The following outlines detailed step taken to achieve this:


### Refactoring The Codes Using Module
#### Created a folder called modules
#### The following folders were created inside the modules folder to combine resources of the similar type: ALB, VPC, Autoscaling, Security, EFS, RDS, Compute
#### Creating the following files for each of the folders: main.tf, variables.tf and output.tf
#### - PBL Folder Structure
<img width="190" alt="pbl structure" src="https://user-images.githubusercontent.com/112771723/204024767-a98ee41c-62bb-4fd6-a7ba-3026b59eece1.png">

#### - Refactoring the code for ALB folder:
