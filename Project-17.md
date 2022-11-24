## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM
### In continuation of project 16, the remaining resources are created in this project in order to set up a secured infrastructure with Terraform
#### STEP 1: Creating Private Subnet
```
# Create Private Subnet 
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
```
<img width="473" alt="private subnet created" src="https://user-images.githubusercontent.com/112771723/203789764-de9c4bdc-bc27-4e7d-b96f-134a7c30bc3d.png">
#### Creating tags attached to both the private and public subnets
```
tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    }
  )  
}
```
<img width="457" alt="tags" src="https://user-images.githubusercontent.com/112771723/203792792-c14850fa-c656-46d2-b51e-dc9cc877054c.png">

### STEP 2: Creating Nat-Gatway
#### First, an elastic ip would be created, which will be associated with the Nat-Gatway
