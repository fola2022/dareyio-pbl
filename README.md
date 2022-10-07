## LAMP STACK IMPLEMENTATION
### Step 1. Installing apache2
#### Command: sudo apt install apache2
![Installing Apache (2)](https://user-images.githubusercontent.com/112771723/188878330-90bf4017-030d-47cd-80ba-c0487383f530.png)

### Verifying the apache2 running
#### Command: sudo systemctl status apache2
![Apache running (2)](https://user-images.githubusercontent.com/112771723/188878610-951a2a90-16ec-43a1-8dbf-49e2339947a5.png)

### Step 2. Installing mysql-server
#### Command: sudo apt install mysql-server
![installing mysql](https://user-images.githubusercontent.com/112771723/188880461-8e6735fc-d783-45ff-8d00-fd792ddc4377.png)

### Logged into the mqsql
#### Command: sudo mysql
![installing mysql2 (2)](https://user-images.githubusercontent.com/112771723/188882398-e209320c-8e4c-4431-83bb-e1e6e946c40b.png)

### Step 3. Interactive script and root user password changed
#### Command: sudo mysql-secure-installation
![installing mysql1 (3)](https://user-images.githubusercontent.com/112771723/188882559-716fa213-34fb-49b9-b017-b4caf9586a5f.png)

![root password (2)](https://user-images.githubusercontent.com/112771723/188882694-8e31ccbe-bda7-4020-9538-e85653094b13.png)

### Step 4. Installing php
#### Command: sudo apt install php libapache2-mod-php php-mysql
![INstalling php (2)](https://user-images.githubusercontent.com/112771723/188883359-1272578b-db18-44dd-b4b4-c782012dac8f.png)

### Step 5. Setup VirtualHost on Apache to Serve PHP
#### Commands:
#### sudo mkdir /var/www/projectlamp (Create project directory)
#### sudo chown -R $USER:$USER /var/www/projectlamp (Change user ownerships)
#### sudo vim /etc/apache2/sites-available/projectlamp.conf (Create Virtual Host conf for project)
#### sudo a2ensite projectlamp (Enable project)
#### sudo a2dissite 000-default (Disable default site)
#### sudo apache2ctl configtest (Test configurations)
#### sudo systemctl reload apache2 (Reload Apache2 service)
![Installing php 2 (2)](https://user-images.githubusercontent.com/112771723/188884486-a63ed49d-78cc-4c85-acb5-f5b51ed47032.png)
![projectdevops](https://user-images.githubusercontent.com/112771723/188884612-d9617511-e9cf-4a98-b12d-5c33e499ca25.png)

#### Checking 


