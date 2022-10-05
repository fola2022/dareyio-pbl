## CONFIGURing APACHE AS A LOAD BALANCER
### Installing Apache Load Balancer on an EC2 instance and configuring it to point traffic coming to load balancer for both Web Servers
##### Commands:
```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```
<img width="521" alt="1 apt update" src="https://user-images.githubusercontent.com/112771723/194092451-e6bafbec-0713-4154-a6b8-d8b1f082a0f9.png">
<img width="517" alt="apache installed" src="https://user-images.githubusercontent.com/112771723/194092517-fd87bf3e-53c2-43e5-8f94-398ab550661c.png">
<img width="512" alt="apache 3" src="https://user-images.githubusercontent.com/112771723/194092573-70408f4a-6738-4e4d-aa5e-6bf1c307e5e7.png">

###  The following modules were enabled
##### Commands:
```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```
<img width="407" alt="apache modules" src="https://user-images.githubusercontent.com/112771723/194092822-c9adb55d-a578-4cec-b23b-99f233e388de.png">
<img width="446" alt="apache module last" src="https://user-images.githubusercontent.com/112771723/194092855-79764f42-d7c7-4131-9e08-d8e84bf04381.png">

#### Making sure apache is up and running
##### Commands:
```
sudo systemctl restart apache2
sudo systemctl status apache2
```
<img width="521" alt="apache running use this" src="https://user-images.githubusercontent.com/112771723/194093190-214d89bb-b166-4707-b059-330ddb05c48b.png">

### Configuring load balancer
##### Commands
```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server
sudo systemctl restart apache2
```
### Verifying that the configuration work on broswer
<img width="569" alt="Annotation 2022-10-05 131232" src="https://user-images.githubusercontent.com/112771723/194092282-0be301db-14da-4128-9d72-f907041b03ee.png">




