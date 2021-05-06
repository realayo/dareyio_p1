# Client-Server Architecture with MySQL
## TASK - Implement a Client Server Architecture using MySQL Database Management System (DBMS)
1. We create and configure two linux virutal servers on AWS and name them accordingly, ``client`` and ``server``.
1. On linux ``server`` instance, install mysql server software
```
sudo apt update
```
```
sudo apt install mysql-server -y
```
3. configure mysql by running a security script 
```
sudo mysql_secure_installation
```
follow the on screen prompts to set up password and login into mysql 
```
sudo mysql
```
4. Create a dedicated user ``project_user`` and grant privileges. 
```
CREATE USER 'project_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
5. Create a database named ``testDB``
```
CREATE DATABASE testDB;
```
6. Grant pivileges to user on the database we created.
```
GRANT ALL PRIVILEGES ON testDB.* TO 'project_user'@'%';
```
![](https://user-images.githubusercontent.com/18899718/117294314-1cd42980-ae38-11eb-9795-e374c8b68414.png)

### Install MySQL client software on client server

1. On ``client`` linux instance, install mysql client software
```
sudo apt update
```
```
sudo apt install mysql-client -y
```

1. We need to update our inbound security rules on the ``server`` instance to accept only connections from our ``client`` server. Under the ``connection type`` we select ``MySQL/Aurora`` and for IP Address we use the client server's private IP addresss. This can be gotten by running ``ifconfig`` command on the client server. 

## Configure MySQL server to accept connections
1. Run the following command to edit mysql-server connection settings. 
```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```
2. Search for ``bind-address`` and replace ``127.0.0.1`` with ``0.0.0.0``
save and exit
3. Restart mysql-daemon using 
```
sudo systemctl restart mysql
```
## Connect to MySQL server from MySQL client
1. We can connect by running 
```
sudo mysql -u project_user -p -h 172.31.4.13
```
2. Enter mysql server password when prompted
1. Run 
```
SHOW DATABASES;
```
![](https://user-images.githubusercontent.com/18899718/117300162-e51cb000-ae3e-11eb-8ed1-5a2d8cf113de.png)