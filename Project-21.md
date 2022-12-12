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

