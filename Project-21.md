## ORCHESTRATING CONTAINERS ACROSS MULTIPLE VIRTUAL SERVERS WITH KUBERNETES. PART 1
### aws installed
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
<img width="512" alt="aws install" src="https://user-images.githubusercontent.com/112771723/206992150-ac80f5f8-6fb9-4f1b-8b52-4a21eced565b.png">

#### aws configured with my access key and secret key, using the command `aws configure'
### kubectl installed
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
```
#### To make it executable
```
chmod +x kubectl
```
#### Move to Bin directory
```
sudo mv kubectl /usr/local/bin/
```
#### Verifying kubectl installation
```
kubectl version --client
```
<img width="946" alt="kubectl install" src="https://user-images.githubusercontent.com/112771723/206995511-061a6b7a-2460-4f74-a745-71ef7683bfdc.png">

### Install CFSSL and CFSSLJSON
```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```
```
chmod +x cfssl cfssljson
```
```
sudo mv cfssl cfssljson /usr/local/bin/
```
<img width="946" alt="kubectl install" src="https://user-images.githubusercontent.com/112771723/206996127-87c4e16f-8d05-463c-b24f-0e73d9c5dcbb.png">

### AWS CLOUD RESOURCES FOR KUBERNETES CLUSTER
#### Step 1 – Configure Network Infrastructure
#### A directory named k8s-cluster-from-ground-up was created
#### Creating Virtual Private Cloud – VPC and storing the ID has variables
```
VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--output text --query 'Vpc.VpcId'
)
```
#### Tagging the VPC
```
NAME=k8s-cluster-from-ground-up

aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```
<img width="508" alt="vpc and name tag" src="https://user-images.githubusercontent.com/112771723/207021776-140e545b-9728-465a-a920-59958db07615.png">

#### Enabling DNS support for VPC
```
aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-support '{"Value": true}'
```
#### Enabling DNS support for hostname
```
aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-hostnames '{"Value": true}'
```
#### Setting the required region
```
AWS_REGION=eu-central-1
```
<img width="497" alt="vpc region" src="https://user-images.githubusercontent.com/112771723/207021848-bd6aab86-f743-419f-b730-c3027f2812a0.png">
<img width="710" alt="vpc" src="https://user-images.githubusercontent.com/112771723/207022149-088c27ce-78bc-41c7-8bbe-27ea63fc3387.png">

#### Configuring DHCP Options Set
```
DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options \
  --dhcp-configuration \
    "Key=domain-name,Values=$AWS_REGION.compute.internal" \
    "Key=domain-name-servers,Values=AmazonProvidedDNS" \
  --output text --query 'DhcpOptions.DhcpOptionsId')
```
#### Tagging the DHCP Option set
```
aws ec2 create-tags \
  --resources ${DHCP_OPTION_SET_ID} \
  --tags Key=Name,Value=${NAME}
```
#### Associating DHCP Option set with VPC
```
aws ec2 associate-dhcp-options \
  --dhcp-options-id ${DHCP_OPTION_SET_ID} \
  --vpc-id ${VPC_ID}
```
<img width="710" alt="dhcp vpc" src="https://user-images.githubusercontent.com/112771723/207024760-5d423116-9ded-41e3-ab0d-429fef36eb54.png">

#### Creating subnet
```
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')
```
```
aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
```
#### Creating the Internet Gateway and attaching to the VPC
```
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')
aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
```  
```
aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}
```
#### Route Table
#### Creating route tables, associating the route table to subnet, and creat a route to allow external traffic to the Internet through the Internet Gateway
```
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --output text --query 'RouteTable.RouteTableId')
aws ec2 create-tags \
  --resources ${ROUTE_TABLE_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 associate-route-table \
  --route-table-id ${ROUTE_TABLE_ID} \
  --subnet-id ${SUBNET_ID}
aws ec2 create-route \
  --route-table-id ${ROUTE_TABLE_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${INTERNET_GATEWAY_ID}
 ```
 <img width="752" alt="subnet to route" src="https://user-images.githubusercontent.com/112771723/207026589-5cfb3118-df06-40b5-9aa5-440f51f12b37.png">
 
 ### Configuring the Security Group
 ```
 # Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}

# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
```
<img width="647" alt="sg1" src="https://user-images.githubusercontent.com/112771723/207047372-97f2beae-614f-475c-8973-0f673ec24992.png">
<img width="632" alt="sg2" src="https://user-images.githubusercontent.com/112771723/207047402-09bccf97-019a-4a3c-8cf2-a788d08fdf4f.png">

#### Creating Network Load Balancer
```
Create a network Load balancer,
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
--name ${NAME} \
--subnets ${SUBNET_ID} \
--scheme internet-facing \
--type network \
--output text --query 'LoadBalancers[].LoadBalancerArn')
```
#### Creating Target Group
```
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
```
#### Registra Target
```
aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}
```
#### Creating a listener to listen for requests and forward to the target nodes on TCP port 6443
```
aws elbv2 create-listener \
--load-balancer-arn ${LOAD_BALANCER_ARN} \
--protocol TCP \
--port 6443 \
--default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
--output text --query 'Listeners[].ListenerArn'
```
#### To get the Kubernetes Public address
```
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
--load-balancer-arns ${LOAD_BALANCER_ARN} \
--output text --query 'LoadBalancers[].DNSName')
```
<img width="731" alt="target  group" src="https://user-images.githubusercontent.com/112771723/207047289-61d26ed7-73e5-4a21-92d8-027421790ec9.png">


 
