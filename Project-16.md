## Automate Infrastructure With IAC using Terraform Part 1
### AWS CLI and Terraform were installed. The terraform was added to path on window
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
        default = "eu-central-1"
    }

    provider "aws" {
        region = var.region
    }
 ```
 #### Doing the same to cidr value in the VPC block
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
