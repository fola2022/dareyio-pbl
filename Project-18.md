## AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3 – REFACTORING
### In continuation to [Project 17](https://github.com/fola2022/dareyio-pbl/blob/main/Project-17.md), the entire code is refactored in order to simplify the code using a Terraform tool called Module.
### The following outlines detailed step taken to achieve this:
### STEP 1: Refactoring the code the root main.tf folder:
<img width="671" alt="root main" src="https://user-images.githubusercontent.com/112771723/204162970-c46d4bbf-aec2-4d71-a97d-13762087c043.png">
<img width="668" alt="root main 2" src="https://user-images.githubusercontent.com/112771723/204162976-9cd57585-0c26-4490-a1da-aa894b935468.png">
<img width="674" alt="root main 3" src="https://user-images.githubusercontent.com/112771723/204162984-1166d61f-ad5b-4560-9095-dbec17104511.png">

#### S3 resource and dynamo resource were added to the main.tf in the root module
#### S3 Resource
```
resource "aws_s3_bucket" "terraform-state" {
  bucket = "fola-terraform18-bucket"
  force_destroy = true
}
resource "aws_s3_bucket_versioning" "version" {
  bucket = aws_s3_bucket.terraform-state.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "first" {
  bucket = aws_s3_bucket.terraform-state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```
#### DynamoDB Resource
```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

#### - Creating a file called Backend.tf and entering the below code:
```
terraform {
  backend "s3" {
    bucket         = "fola-terraform18-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### Refactoring The Codes Using Module
#### Created a folder called modules
#### The following folders were created inside the modules folder to combine resources of the similar type: ALB, VPC, Autoscaling, Security, EFS, RDS, Compute
#### Creating the following files for each of the folders: main.tf, variables.tf and output.tf
#### - PBL Folder Structure
<img width="190" alt="pbl structure" src="https://user-images.githubusercontent.com/112771723/204024767-a98ee41c-62bb-4fd6-a7ba-3026b59eece1.png">

### Refactoring the code for VPC folder:
#### variable.tf
<img width="684" alt="variable vpc" src="https://user-images.githubusercontent.com/112771723/204163290-4db4a11e-f03c-473c-9e25-38f923dde78b.png">
<img width="604" alt="variable vpc 2" src="https://user-images.githubusercontent.com/112771723/204163304-b4ed53b0-9635-4160-b504-a230f80b90be.png">

#### output.tf
<img width="645" alt="output vpc" src="https://user-images.githubusercontent.com/112771723/204163336-2c4ee91f-0283-422e-a5b5-b167faa7b4ee.png">
<img width="372" alt="output vpc 2" src="https://user-images.githubusercontent.com/112771723/204163338-52f0da93-993d-4db6-b4fb-7b720e7e0e7d.png">

### - Refactoring the code for ALB folder:
#### variable.tf
<img width="645" alt="variable alb" src="https://user-images.githubusercontent.com/112771723/204162150-d43b331e-2fc3-48c4-857b-814de7cbafc3.png">
<img width="606" alt="variable alb 2" src="https://user-images.githubusercontent.com/112771723/204162162-de331ac0-9b34-432b-8ac1-c01acf0eaaca.png">

#### Output.tf
<img width="546" alt="output" src="https://user-images.githubusercontent.com/112771723/204162037-72c401b4-ace6-4023-a477-52c1ab3f3223.png">

### Refactoring the code for Autoscaling folder:
#### variables.tf
<img width="570" alt="variable autoscaling" src="https://user-images.githubusercontent.com/112771723/204162296-4a992f57-6420-4679-b784-90bae8dcf1b1.png">
<img width="640" alt="variable autoscaling 2" src="https://user-images.githubusercontent.com/112771723/204162315-f2f9b5c5-6fcc-42d4-b03a-3952e43df89e.png">

### Refactoring the code for security folder:
#### variable.tf
<img width="525" alt="variable security" src="https://user-images.githubusercontent.com/112771723/204162624-51456449-3c6f-4d0e-a1ba-0922a582876e.png">

#### outputs.tf
<img width="598" alt="output security" src="https://user-images.githubusercontent.com/112771723/204162397-947bf8b6-9836-412d-a7da-07e11c171fdd.png">

### Refactoring the code for EFS folder:
#### variables.tf
<img width="653" alt="variable efs" src="https://user-images.githubusercontent.com/112771723/204162667-9c102394-5ab6-498d-be0a-1ffa38ea77ba.png">

### Refactoring the code for RDS folder:
#### variables.tf
<img width="608" alt="variable rds" src="https://user-images.githubusercontent.com/112771723/204162710-44699e31-e71c-478b-941d-702fdd63a9ee.png">

### STEP 2: Executing The Terraform Plan and Terraform Apply
<img width="502" alt="apply1" src="https://user-images.githubusercontent.com/112771723/204163431-1309076c-2840-428d-b5fe-c8b3d9c8a178.png">
<img width="754" alt="apply2" src="https://user-images.githubusercontent.com/112771723/204163436-986c4a70-b4e4-4184-bac9-9b6b5796cc14.png">
<img width="944" alt="last" src="https://user-images.githubusercontent.com/112771723/204163743-69bee233-fe32-4fe4-a456-94ddd3a2a75e.png">

### STEP 3: Configuring A Backend On The S3 Bucket Created
#### By default the Terraform state is stored locally, to store it remotely on AWS using S3 bucket as the backend and also making use of DynamoDB as the State Locking the following setup is done:
#### - Creating a file called Backend.tf and entering the following code:
```
terraform {
  backend "s3" {
    bucket         = "fola-terraform18-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
#### Then terraform init was run to initialize the backend.tf file
<img width="467" alt="backend" src="https://user-images.githubusercontent.com/112771723/204163762-4214a481-f8f6-4be7-b8b9-065a27fd1490.png">




