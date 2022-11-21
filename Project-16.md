## Automate Infrastructure With IAC using Terraform Part 1
### USING AWS CLI
#### Installing Terraform
##### Commands:
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform
```
<img width="421" alt="terraform install 1" src="https://user-images.githubusercontent.com/112771723/203073232-2ff0fd1f-b96e-45c0-a3cf-9b0de38a5a5e.png">
<img width="412" alt="terraform install 2" src="https://user-images.githubusercontent.com/112771723/203073300-8b410539-44a8-435f-953c-3219e4e32fb2.png">
<img width="425" alt="Terraform install 3" src="https://user-images.githubusercontent.com/112771723/203073334-d528517f-5ff4-4167-bda0-d3648e9d8743.png">

#### Verifying the installation
```
terraform --help
``
<img width="410" alt="verify terraform" src="https://user-images.githubusercontent.com/112771723/203073403-20cef8c0-fb95-4a66-9d8c-b0185a1e421d.png">

#### Installing Tree
```
sudo yum install tree
```
<img width="607" alt="tree install" src="https://user-images.githubusercontent.com/112771723/203077599-e8e6e3b4-809d-4a3a-b34f-007dc7a4f5dc.png">

#### Initializing Terraform
```
terraform init
```
<img width="504" alt="terraform init successful" src="https://user-images.githubusercontent.com/112771723/203077687-af240fb9-c8b8-4761-b559-972a72484f4b.png">
<img width="185" alt="terraform init added file" src="https://user-images.githubusercontent.com/112771723/203077731-57f88b35-0f61-4f41-ab32-4f9dc1dca6de.png">
