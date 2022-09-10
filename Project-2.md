### Step 1. Installing nginx web server
#### - Command: sudo apt install nginx
<img width="482" alt="installing nginx" src="https://user-images.githubusercontent.com/112771723/189488337-a68cc254-8cfc-4758-a51c-45542b99537a.png">

### Verifying that nginx was successfully installed
#### - Command: sudo systemctl status nginx
<img width="509" alt="nginx running" src="https://user-images.githubusercontent.com/112771723/189488366-1725d239-ab77-48d0-80af-514445e475e0.png">
<img width="484" alt="ngnix 80tcp" src="https://user-images.githubusercontent.com/112771723/189488386-23bb5b13-c5c5-40a8-af61-8321678fec3c.png">

### Step 2. Installing Mysql
#### - Command: sudo apt install mysql-server
####            sudo mysql
<img width="489" alt="sudo mysql" src="https://user-images.githubusercontent.com/112771723/189488677-88bebd1a-0ea7-4801-9757-ff9b3bc18e3a.png">

### Interactive scripting running
#### - Command: sudo mysql_secure_installation
####            sudo mysql -p
<img width="506" alt="mysql scripting and password" src="https://user-images.githubusercontent.com/112771723/189489262-27bfa988-b99b-416f-961e-f6ab3576bd0f.png">
<img width="496" alt="mysql -p" src="https://user-images.githubusercontent.com/112771723/189489274-111238df-e1ab-4aed-85dd-d4d987be73ea.png">

### Step 3. Installing PHP
#### - Command: sudo apt instal php-fpm php-mysql
<img width="490" alt="php install" src="https://user-images.githubusercontent.com/112771723/189489377-da15dc67-8388-4529-8819-e97232db3003.png">

### Step 4. Configuring nginx to use PHP processor
#### - Command: sudo mkdir /var/www/projectLEMP
####            sudo chown -R $USER:$USER /var/www/projectLEMP
####            sudo nano /etc/nginx/sites-available/projectLEMP
####            sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
####            sudo nginx -t
<img width="514" alt="nginx conf" src="https://user-images.githubusercontent.com/112771723/189489603-22547110-5acb-4672-8e16-9cbad102f0e7.png">

### Step 5: Testing PHP with nginx
### - Command: sudo nano /var/www/projectLEMP/info.php
<img width="719" alt="php" src="https://user-images.githubusercontent.com/112771723/189489947-676fdc96-dc48-4f13-990e-9a729c0d68ba.png">

### Step 6: Retrieving data from Mysql database with PHP 
#### - Command: sudo mysql
####            mysql> CREATE DATABASE `example_database`;
####            mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
####            mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
####            mysql -u example_user -p
####            mysql> SHOW DATABASES;
<img width="533" alt="databasee" src="https://user-images.githubusercontent.com/112771723/189490210-a5823ca6-6ac4-4ded-8beb-9e148599cb63.png">

### Creating a table named to-do list 
#### - Command: mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
####            mysql>  SELECT * FROM example_database.todo_list;
<img width="474" alt="Table" src="https://user-images.githubusercontent.com/112771723/189490407-2050874e-a62a-49c1-a7bc-b0eeabdba994.png">

### Creating a new PHP that will connect to Mysql
#### - Command: nano /var/www/projectLEMP/todo_list.php
#### The page was then accessed on the web brower
<img width="173" alt="project 2 last page" src="https://user-images.githubusercontent.com/112771723/189490674-9ecc4b3b-819c-4da8-8fe2-a588c823b2c8.png">




