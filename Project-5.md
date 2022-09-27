# CLIENT-SERVER ARCHITECTURE WITH MYSQL
### First, two linux-based virtual servers (EC2 instances in AWS) were created and configured.
#### Installing mysql-server on terminal 1
#### Installing mysql-client on terminal 2
<img width="778" alt="install mysql server" src="https://user-images.githubusercontent.com/112771723/192095077-782f48e4-27f3-47e6-96b6-81e97fb9d89c.png">
<img width="779" alt="install mysql client" src="https://user-images.githubusercontent.com/112771723/192095121-8de4b283-6fdc-4ae9-a5b4-dd43da10c6ec.png">

### Configuring MySQL server to allow connections from remote hosts
##### Commands are;
```
sudo systemcl enable mysql
sudo mysql
sudo mysql_secure_installation
sudo mysql -p
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
<img width="670" alt="systemctl enable mysql" src="https://user-images.githubusercontent.com/112771723/192095967-70aa83b1-2283-4e1a-b48c-bc4f30298e5e.png">
<img width="602" alt="mysql" src="https://user-images.githubusercontent.com/112771723/192095854-1838d76a-56ce-4bfb-96e5-eb5229ffcf64.png">
<img width="701" alt="sudo mysql" src="https://user-images.githubusercontent.com/112771723/192096006-d4fd34e1-a784-4a5f-b098-185c8f744a0a.png">
<img width="704" alt="binding add edit" src="https://user-images.githubusercontent.com/112771723/192096123-8356b78c-1706-4679-966b-698ca54fe5f2.png">

#### Successfully connected to a remote MySQL server
 <img width="503" alt="database" src="https://user-images.githubusercontent.com/112771723/192096201-785bd404-ea0c-45d8-b58c-77a730b35883.png">





