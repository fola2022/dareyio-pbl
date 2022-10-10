## CONFIGURING NGINX AS A LOAD BALANCER
### STEP 1: INSTALLING NGINX 
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

### STEP 2: REGISTERING A NEW DOMAIN NAME AND CONFIGURING SECURED CONNECTION USING SSL/TLS CERTIFICATES
#### A new free domain was registered on Freenom website. The domain name is "Updevops.tk" "www.updevops.tk"
#### In route 53 on AWS a record was created to Nginx loader balancer using Elastic IP address
<img width="491" alt="route53" src="https://user-images.githubusercontent.com/112771723/194895685-bcfc23c2-c098-4ca6-b245-f51ef3df5144.png">
<img width="476" alt="route" src="https://user-images.githubusercontent.com/112771723/194895703-e7708c27-6381-4172-9583-ec50dd6eb340.png">

#### Checking that web server can be reached from broswer using the domain name using HTTP protocol
<img width="697" alt="browers" src="https://user-images.githubusercontent.com/112771723/194896318-4511e683-8093-4393-90d0-84addfd71666.png">

#### Configuring Nginx to recognize domain name
#### Installing certbot
##### Command:
```
sudo apt install certbot -y
sudo apt install python3-certbot-nginx -y
```
<img width="497" alt="certbot installed" src="https://user-images.githubusercontent.com/112771723/194896995-e26e25e2-414c-4b4e-b27e-254d2983e281.png">
<img width="485" alt="python3 certbot module-dependency" src="https://user-images.githubusercontent.com/112771723/194897389-69744ac8-8846-4026-b51f-5fd6e650d8fb.png">

#### The SSL certificate
##### Command:
```
sudo certbot --nginx -d updevops.tk -d www.updevops.tk
```
<img width="486" alt="website certificate" src="https://user-images.githubusercontent.com/112771723/194898528-c60504bd-63c4-4d56-8fd8-4e6d01b82a29.png">
<img width="484" alt="website cert 2" src="https://user-images.githubusercontent.com/112771723/194898576-8daa2a5e-7c1e-4bd6-93c6-cd9394c0759d.png">

#### Secured on broswer
<img width="757" alt="cert" src="https://user-images.githubusercontent.com/112771723/194898643-06f76451-e841-406d-91e7-ebe9d5357c6c.png">

#### Setting up periodical renewal of SSL/TLS certificate
#### The crontab file was edited
##### Command:
```
crontab -e
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
<img width="467" alt="ssl renewal" src="https://user-images.githubusercontent.com/112771723/194899329-0fa465c0-6e8e-468d-aa82-2d5e09b4f260.png">





