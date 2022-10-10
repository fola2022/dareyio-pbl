## CONFIGURING NGINX AS A LOAD BALANCER
### Install Nginx
#### Command:
```
sudo apt update
sudo apt install nginx
```
<img width="490" alt="apt update and nginx installed" src="https://user-images.githubusercontent.com/112771723/194891491-b2475ae6-bcb4-470f-8cf8-3e561626ebdc.png">

#### Configure Nginx load balancer using Web Servers’ names defined in /etc/hosts
#### Commands:
```
sudo vi /etc/hosts
sudo vi /etc/nginx/nginx.conf
sudo nginx -t
```
```
upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
  ```
<img width="466" alt="nginx conf" src="https://user-images.githubusercontent.com/112771723/194892159-6ae4ad46-5589-4ebd-9f73-d4fc6ff93ecd.png">
<img width="486" alt="nginx syntax reload" src="https://user-images.githubusercontent.com/112771723/194893027-24144ef3-3d46-4329-9f0c-8ae576414967.png">

### Restarting Nginx and making sure it up and running
#### Commands:
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
<img width="491" alt="ngnix start and running" src="https://user-images.githubusercontent.com/112771723/194892730-21f7e77a-527b-4789-a0e1-2c03bb75d5c9.png">






