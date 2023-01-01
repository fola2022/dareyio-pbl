# BUILDING ELASTIC KUBERNETES SERVICE (EKS) WITH TERRAFORM
## INTRODUCTION
### In this project, the Elastic Kubernates Service (EKS), which is a managed kubernetes service that takes care of undifferentiated heavy lifting involved in setting up Kubernetes, is provisioned using Terraform. And further in the project, Helm which is a package manager for Kubernetes is used to deploy multiple applications.
### STEP 1: Configuring The Terraform Module For EKS
#### Creating AWS S3 bucket from a CLI to store the Terraform state:
```
aws s3api create-bucket --bucket eks-terraform-buck --region us-east-1
```
<img width="583" alt="s3 bucket created" src="https://user-images.githubusercontent.com/112771723/210172793-5d06514e-2800-4c13-b1c2-72492d0c4fc2.png">

#### Creating a new directory eks for the Terraform file
#### Creating a file backend.tf for the remote state in S3:
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

terraform {
  backend "s3" {
    bucket         = "eks-terraform-bucket24"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
<img width="398" alt="backend tf" src="https://user-images.githubusercontent.com/112771723/210172873-4b2f9f32-23f5-4665-bd11-c288922792ae.png">

#### Creating the network.tf file to provision Elastic IP for Nat Gateway, VPC, Private and public subnets:
```
# reserve Elastic IP to be used in our NAT gateway
resource "aws_eip" "nat_gw_elastic_ip" {
  vpc = true

  tags = {
    Name            = "${var.cluster_name}-nat-eip"
    iac_environment = var.iac_environment_tag
  }
}

module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "${var.name_prefix}-vpc"
  cidr = var.main_network_block
  azs  =  [
    "us-east-1a",
    "us-east-1b",
    "us-east-1c",
    "us-east-1d",
    "us-east-1f",
  ]

  private_subnets = [
    # this loop will create a one-line list as ["10.0.0.0/20", "10.0.16.0/20", "10.0.32.0/20", ...]
    # with a length depending on how many Zones are available
    for zone_id in data.aws_availability_zones.available_azs.zone_ids :
    cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) - 1)
  ]

  public_subnets = [
    for zone_id in data.aws_availability_zones.available_azs.zone_ids :
    cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) + var.zone_offset - 1)
  ]
  enable_nat_gateway     = true
  single_nat_gateway     = true
  one_nat_gateway_per_az = false
  enable_dns_hostnames   = true
  reuse_nat_ips          = true
  external_nat_ip_ids    = [aws_eip.nat_gw_elastic_ip.id]

  # Add VPC/Subnet tags required by EKS
  tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    iac_environment                             = var.iac_environment_tag
  }
  public_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                    = "1"
    iac_environment                             = var.iac_environment_tag
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"           = "1"
    iac_environment                             = var.iac_environment_tag
  }
}
```
<img width="845" alt="network tf" src="https://user-images.githubusercontent.com/112771723/210186405-2c69d309-91c2-4569-9209-6535cd55b9f1.png">

#### Creating the variables.tf file
```
# create some variables
variable "cluster_name" {
  type        = string
  description = "EKS cluster name."
}

variable "iac_environment_tag" {
  type        = string
  description = "AWS tag to indicate environment name of each infrastructure object."
}

variable "name_prefix" {
  type        = string
  description = "Prefix to be used on each infrastructure object Name created in AWS."
}

variable "main_network_block" {
  type        = string
  description = "Base CIDR block to be used in our VPC."
}

variable "subnet_prefix_extension" {
  type        = number
  description = "CIDR block bits extension to calculate CIDR blocks of each subnetwork."
}

variable "zone_offset" {
  type        = number
  description = "CIDR block bits extension offset to calculate Public subnets, avoiding collisions with Private subnets."
}

variable "admin_users" {
  type        = list(string)
  description = "List of Kubernetes admins."
}

variable "developer_users" {
  type        = list(string)
  description = "List of Kubernetes developers."
}

variable "asg_instance_types" {
  description = "List of EC2 instance machine types to be used in EKS."
}

variable "autoscaling_minimum_size_by_az" {
  type        = number
  description = "Minimum number of EC2 instances to autoscale our EKS cluster on each AZ."
}

variable "autoscaling_maximum_size_by_az" {
  type        = number
  description = "Maximum number of EC2 instances to autoscale our EKS cluster on each AZ."
}
```
<img width="785" alt="variable tf" src="https://user-images.githubusercontent.com/112771723/210186441-4883a461-eeab-4682-8e72-be515b47b628.png">

#### Creating the data.tf file that will pull the available AZs for use:
```
# get all available AZs in our region
data "aws_availability_zones" "available_azs" {
  state = "available"
}
data "aws_caller_identity" "current" {} # used for accesing Account ID and ARN

# get EKS cluster info to configure Kubernetes and Helm providers
data "aws_eks_cluster" "cluster" {
  name = module.eks_cluster.cluster_id
}
data "aws_eks_cluster_auth" "cluster" {
  name = module.eks_cluster.cluster_id
}
```
#### Creating the eks.tf file that will provision EKS cluster
```
module "eks_cluster" {
  source                          = "terraform-aws-modules/eks/aws"
  version                         = "~> 18.0"
  cluster_name                    = var.cluster_name
  cluster_version                 = "1.22"
  vpc_id                          = module.vpc.vpc_id
  subnet_ids                      = module.vpc.private_subnets
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = true

  # Self Managed Node Group(s)
  self_managed_node_group_defaults = {
    instance_type                          = var.asg_instance_types[0]
    update_launch_template_default_version = true
  }
  self_managed_node_groups = local.self_managed_node_groups

  # aws-auth configmap
  create_aws_auth_configmap = true
  manage_aws_auth_configmap = true
  aws_auth_users            = concat(local.admin_user_map_users, local.developer_user_map_users)
  tags = {
    Environment = "prod"
    Terraform   = "true"
  }
}
```
<img width="605" alt="eks cluster eks tf" src="https://user-images.githubusercontent.com/112771723/210186423-269fe913-cbee-4ff7-981e-2c32cbf2038f.png">

#### Creating the locals.tf file for local variables because Terraform does not allow assigning variable to variables
```
# render Admin & Developer users list with the structure required by EKS module
locals {
  admin_user_map_users = [
    for admin_user in var.admin_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${admin_user}"
      username = admin_user
      groups   = ["system:masters"]
    }
  ]
  developer_user_map_users = [
    for developer_user in var.developer_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${developer_user}"
      username = developer_user
      groups   = ["${var.name_prefix}-developers"]
    }
  ]

  self_managed_node_groups = {
    worker_group1 = {
      name = "${var.cluster_name}-wg"

      min_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      desired_size  = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      max_size      = var.autoscaling_maximum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      instance_type = var.asg_instance_types[0].instance_type

      bootstrap_extra_args = "--kubelet-extra-args '--node-labels=node.kubernetes.io/lifecycle=spot'"

      block_device_mappings = {
        xvda = {
          device_name = "/dev/xvda"
          ebs = {
            delete_on_termination = true
            encrypted             = false
            volume_size           = 10
            volume_type           = "gp2"
          }
        }
      }

      use_mixed_instances_policy = true
      mixed_instances_policy = {
        instances_distribution = {
          spot_instance_pools = 4
        }

        override = var.asg_instance_types
      }
    }
  }
}
```
#### Creating the terraform.tfvars to set values for variables
```
cluster_name            = "tooling-app-eks"
iac_environment_tag     = "development"
name_prefix             = "somex-io-eks"
main_network_block      = "10.0.0.0/16"
subnet_prefix_extension = 4
zone_offset             = 8

# Ensure that these users already exist in AWS IAM. Another approach is that you can introduce an iam.tf file to manage users separately, get the data source and interpolate their ARN.
admin_users                    = ["somex", "grandol"]
developer_users                = ["alex", "victor"]
asg_instance_types             = [{ instance_type = "t3.small" }, { instance_type = "t2.small" }, ]
autoscaling_minimum_size_by_az = 1
autoscaling_maximum_size_by_az = 5
```
#### Creating the provider.tf file
```
provider "aws" {
  region = "us-west-1"
}

provider "random" {
}

# get EKS authentication for being able to manage k8s objects from terraform
provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}
```
#### Running the terraform init command
<img width="719" alt="terraform init 1" src="https://user-images.githubusercontent.com/112771723/210186753-d68a61dd-d325-4ade-9e08-7feb8a8e8c68.png">
<img width="479" alt="terraform init 2" src="https://user-images.githubusercontent.com/112771723/210186760-75a0bd9a-abb1-4447-8e59-2ae769ea2a9c.png">

#### Running the terraform plan command to confirm the configuration
<img width="899" alt="terraform plan 1" src="https://user-images.githubusercontent.com/112771723/210186771-fe09aa9c-a527-4b96-87af-4de47722de80.png">

#### Running the terraform apply command
<img width="945" alt="terraform apply" src="https://user-images.githubusercontent.com/112771723/210187680-87981ea4-e5d4-4532-9a7a-b5b3e28114f6.png">

#### Creating the kubeconfig file using awscli
```
aws eks update-kubeconfig --name tooling-app-eks --region us-west-1 --kubeconfig kubeconfig
```
### STEP 2: Installing Helm From Script
```
wget https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz
tar xvf helm-v3.6.3-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin
rm -rf linux-amd64
helm version
```
<img width="941" alt="helm install 1" src="https://user-images.githubusercontent.com/112771723/210186869-2f899c85-d007-4281-9d79-c5e3447c389d.png">

### STEP 3: Deploying Jenkins With Helm
Adding the Jenkins' repository to helm so it can be easily downloaded and deployed:
```
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins --kubeconfig kubeconfig
```
<img width="946" alt="install jenkins" src="https://user-images.githubusercontent.com/112771723/210187004-e3c850fc-47c9-4fcd-bce3-3bf836f6f1d9.png">

#### In order to run the kubectl commands without specifying the kubeconfig file, a package manager for kubectl called krew is installed so that it will enable us to install plugins to extend the functionality of kubectl:
<img width="717" alt="krew install 1" src="https://user-images.githubusercontent.com/112771723/210187224-68ef22f4-ff4f-4581-b921-dffb1897caf6.png">
<img width="812" alt="krew install 2" src="https://user-images.githubusercontent.com/112771723/210187230-0b959c56-ec63-44ae-96f6-f1e070b5bf22.png">
<img width="557" alt="install knew 2" src="https://user-images.githubusercontent.com/112771723/210187149-0d960e50-2fab-44d5-84dd-15621e0bdf16.png">

#### Installing Konfig plugin:
```
kubectl krew install konfig
```
<img width="479" alt="konfig install" src="https://user-images.githubusercontent.com/112771723/210187196-03fe72ec-ec17-4592-a1a5-74b472ceb2dc.png">

#### Running some commands to inspect the Jenkins installation and Accessing the Jenkins app from the browser:http://localhost:8080
<img width="483" alt="jenkins running" src="https://user-images.githubusercontent.com/112771723/210187045-13009a4d-5f97-4c20-a324-a305fe2deaa1.png">
<img width="926" alt="end" src="https://user-images.githubusercontent.com/112771723/210187079-cc74bdf9-0872-4ced-9c6b-525e15f3fe10.png">

#### STEP 4: Deploying Artifactory With Helm
```
helm repo add jfrog https://charts.jfrog.io 
helm repo update
helm upgrade --install artifactory jfrog/artifactory
```
<img width="833" alt="artifactory" src="https://user-images.githubusercontent.com/112771723/210187317-3fe82b8a-4ad2-40d2-a958-2ed8c9464d0e.png">

#### STEP 5: Deploying Hashicorp Vault With Helm
```
helm repo add hashicorp https://helm.releases.hashicorp.com 
helm repo update
helm install vault hashicorp/vault
```
<img width="727" alt="hashicorp vault installed" src="https://user-images.githubusercontent.com/112771723/210187357-76c843ea-d6bb-4dd5-bc8b-b16db8ec7b33.png">

#### Port forwarding to access Hashicorp vault from the UI:
```
kubectl port-forward svc/vault 5000:8200
```
<img width="948" alt="hashicorp service" src="https://user-images.githubusercontent.com/112771723/210187374-1792faa2-3c66-4939-9865-8374af7e6ef4.png">

#### Accessing the app from the browser:http://localhost:5000
<img width="830" alt="hashicorp vault" src="https://user-images.githubusercontent.com/112771723/210187393-0e48c35c-9a1e-4342-846c-5a66ecc5072d.png">

#### STEP 6: Deploying Prometheus With Helm
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
```
<img width="875" alt="prometheus installed" src="https://user-images.githubusercontent.com/112771723/210187426-9161cd1d-b40a-47f6-9779-d4f0b46d4d92.png">

#### Port forwarding to access prometheus for alert manager from the UI:$ kubectl port-forward svc/myprometheus-alertmanager 8000:9093
#### Port forwarding to access prometheus for alert manager from the UI:$ kubectl port-forward svc/myprometheus-prometheus-pushgateway 8001:9091

<img width="587" alt="prometheus service" src="https://user-images.githubusercontent.com/112771723/210187436-0341f666-145d-49a3-a631-7dea5a27b887.png">
<img width="893" alt="prometheus" src="https://user-images.githubusercontent.com/112771723/210187467-46b30326-ca31-4c8f-b416-73664728beba.png">
<img width="897" alt="prometheus 2" src="https://user-images.githubusercontent.com/112771723/210187470-c62de5ac-d143-4ae0-8b5b-704374cb666d.png">

#### STEP 6: Deploying Grafana With Helm
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana-tool grafana/grafana
```
<img width="949" alt="grafana installed" src="https://user-images.githubusercontent.com/112771723/210187523-88905ef1-cb86-439d-a4af-c3bdd25f84fb.png">
<img width="946" alt="grafana pod and service" src="https://user-images.githubusercontent.com/112771723/210187541-36af2f4d-5397-4701-a8e3-01ff118e10db.png">

#### Port forwarding to access grafana from the UI:$ kubectl port-forward svc/grafana-tool 8300:80
<img width="686" alt="grafana broswer" src="https://user-images.githubusercontent.com/112771723/210187554-609e4dd6-3a24-4fec-b4a3-cf508dbc7fd3.png">
<img width="905" alt="grafana b 2" src="https://user-images.githubusercontent.com/112771723/210187571-189ca9c3-7745-4df5-b7da-9220431bfa8f.png">

#### STEP 6: Deploying Elasticsearch With Helm
```
helm repo add odysseycloud https://odysseycloud.github.io/oc-charts 
helm repo update
helm install elasticsearch odysseycloud/elasticsearch
```
<img width="899" alt="elasticsearch" src="https://user-images.githubusercontent.com/112771723/210187624-a334ad7f-1235-40f8-be06-6eccf90e4b7b.png">

### All Pods
---
kubectl get pod
```
<img width="518" alt="pod" src="https://user-images.githubusercontent.com/112771723/210187649-28436414-348f-4b0f-bdea-82dccf1cf25f.png">













