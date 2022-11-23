## Automate Infrastructure With IAC using Terraform Part 1
### This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.
### IAM user was created with AdministrativeAccess permissions in AWS and acquired the access key and secret key. Then an S3 bucket named "fola-devv-terraform-bucket" was created in AWS to store Terraform state file
### AWS CLI and Terraform were installed. The access key and security key was used to configure AWS CLI and the terraform was added to path on window
#### Installed AWS CLI and S3 list
```
aws --version
aws s3 ls
```
<img width="346" alt="aws" src="https://user-images.githubusercontent.com/112771723/203593352-970b3afa-b1f3-482f-9696-dad041d34385.png">

#### Installed Terraform
<img width="350" alt="terraform install" src="https://user-images.githubusercontent.com/112771723/203591690-e3bd565a-a557-426a-a17c-8a0e09f8713b.png">

#### Verifying the installation
```
terraform --help
```
<img width="482" alt="tf help" src="https://user-images.githubusercontent.com/112771723/203592417-4af5d177-637b-43d2-a4fa-3a015d68ce03.png">

#### Initializing Terraform
```
terraform init
```
<img width="522" alt="terraform init 1" src="https://user-images.githubusercontent.com/112771723/203592982-ca2c694a-d0a0-4d8e-857e-94d58286c604.png">

#### A folder named PBL was created and main.tf was created in it
#### To create the VPC resource, the main.tf was updated with the below code
```
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```
#### Then terraform plan and apply was run
```
terraform plan
```
```
terraform apply
```
<img width="571" alt="terraform plan 1" src="https://user-images.githubusercontent.com/112771723/203594917-501c869e-375f-48cb-9322-2f97e2cfa044.png">
<img width="403" alt="tf apply" src="https://user-images.githubusercontent.com/112771723/203595880-dca40780-4051-4240-863c-6f8d48339297.png">

#### To create two subnet for the VPC resources created, the below code was added to the main.tf file
```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```
### FIXING THE HARD CODED VALUES BY CODE REFACTORING USING VARIABLES
#### Starting with the provider block
```
    variable "region" {
        default = "us-east-1"
    }

    provider "aws" {
        region = var.region
    }
 ```
 #### Doing the same to cidr value in the VPC block
 ```
 variable "region" {
        default = "us-east-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
 ```
 ### Fixing multiple resource blocks by introducing Loops & Data sources
 #### Fetching Availability zones from AWS, and replacing the hard coded value in the subnet’s availability_zone section with the use of Data Sources.
```
 # Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }
```
### Introducing a count argument in the subnet block to make use of the new data resource
```
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```
#### The count function will tell terraform to invoke a loop to create 2 subnets and the data resource will return a list object that contains a list of AZs. But if Terraform is being run with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because the cidr_block still has to be hard coded because the same cidr_block cannot be created twice within the same VPC.
#### To make the cidr block dynamic a function cidrsubnet() is introduced which accepts 3 parameters.
```
 # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```
#### Introuducing length() function, which simply determines the length of a given list and passing it to data.aws_availability_zones.available.names. But since this returns the value of 5 (us-east-1 have 5 AZ) instead of 2 that is preffered, the variable to store the desired number of public subnets is declared and it is set to the default value.
```
variable "preferred_number_of_public_subnets" {
  default = 2
}
```
#### Updating the count argument with a condition of which Terraform checks first if there is a desired number of subnets, Otherwise, it will use the data returned by the lenght function.
```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```
###  Variables.tf And terraform.tfvars
#### For the code to be more readable and well structured, instead of having a long list of variables in main.tf file, the variable declarations is moved to a separate file and a file for non-default values for each of the variables is created.
#### Creating a new file and naming it variables.tf
#### All variable declarations were moved into the new file.
#### Creating another file and naming it terraform.tfvars
#### Setting values for each of the variables
#### Below is the terraform.tfvars file 
```
region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2
```
#### Below is the variables.tf file
```
variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = null
}
terraform.tfvars
region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2
```
#### main.tf
<img width="614" alt="main" src="https://user-images.githubusercontent.com/112771723/203605606-7bea1019-f693-4483-acd1-8268575a2eca.png">

### The terraform applly created the VPC and subnet on AWS console
<img width="719" alt="vpc on aws console" src="https://user-images.githubusercontent.com/112771723/203605956-1038896d-a745-4df4-83cb-36f1cef0c67e.png">
<img width="713" alt="public subnet " src="https://user-images.githubusercontent.com/112771723/203605975-1ba24184-ae53-4708-b4b0-42255d4006b0.png">


