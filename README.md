# LAMP Stack Implementation in AWS

1. set up an Ubuntu EC2 instance on aws
1. Under Security groups, open TCP port 22 to allow SSH into the instance.
1. Download pem key and 
1. change permissions for the private key file using ```chmod 400 'keyfile'.pem```
1. ssh into the instace using the keyfile, username and public ip address of the instance. ``ssh -i keyfile.pem user@ip-address``
1. Download latest update for the remote computer
![update OS](https://user-images.githubusercontent.com/18899718/113864026-0458ec80-9770-11eb-8efa-02349bebe7f0.png)
1. Install Apache 
![Install apache](https://user-images.githubusercontent.com/18899718/113864165-2e121380-9770-11eb-88c0-d6222cfa3b46.png)
1. Check the status of apache to make sure it's working 
![Apache status](https://user-images.githubusercontent.com/18899718/113865372-9e6d6480-9771-11eb-943f-940513e5733b.png)
1. Enable firewall
![enable firewall](https://user-images.githubusercontent.com/18899718/113865620-e7bdb400-9771-11eb-8881-7fa2b5f0af43.png)
1. check to see make sure apache is responding to ``curl`` request
![curl locahhost](https://user-images.githubusercontent.com/18899718/113866501-07091100-9773-11eb-99da-8129df09eb43.png)
1. Check to to see apache is working on properly on the web
![access apache via public IP](https://user-images.githubusercontent.com/18899718/113866822-61a26d00-9773-11eb-8384-f22f9da83e3c.png)


## Install mysql

1. Install mysql on the server using ``apt`` 
![install mysql](https://user-images.githubusercontent.com/18899718/113880424-168f5680-9781-11eb-9f8d-144ff03a59a5.png)
1. Remove inscure settings but running default security scripts to remove them
![secure mysql](https://user-images.githubusercontent.com/18899718/113880752-5f470f80-9781-11eb-94ce-c49f337b7dbf.png) Provide necessary answers to the screen prompts and then enter a secure password for the mysql
1. When done, login to mysql using ``sudo mysql``
![login](https://user-images.githubusercontent.com/18899718/113881049-adf4a980-9781-11eb-88f2-b11ebf00c5b1.png)
1. Once everything is right, log out of mysql using ``exit``

# Install PHP

1. Install package managers to handle, mysql, apache and other dependencies. 
![install php](https://user-images.githubusercontent.com/18899718/113881714-4be87400-9782-11eb-8ab6-49ee0e7c1175.png)
1. Check current PHP version using ``php -v``
![PHP Version](https://user-images.githubusercontent.com/18899718/113882088-a1248580-9782-11eb-91b2-9bfb56d6e32c.png)

# Create virtual host for website

1. create a directory for our project using *projectlamp* as directory name. 
1. Change ownership of the directory to the current user
1. create and open a configuration file for in apaches's `sites-available` 
![virtual hosts](https://user-images.githubusercontent.com/18899718/113882895-4e979900-9783-11eb-9688-a6f4dddbfcda.png)
1. The configuration in the image below will be pasted in the conf file.
![configuration](https://user-images.githubusercontent.com/18899718/113883114-7e46a100-9783-11eb-816a-374b6c8aa80c.png)
1. Enable the new virtual host
![enalbe virutal host](https://user-images.githubusercontent.com/18899718/113883765-12b10380-9784-11eb-93c1-7ce082fd1b5b.png)
1. Remove default configuration file
![remove host](https://user-images.githubusercontent.com/18899718/113884050-4b50dd00-9784-11eb-9667-cec5e7e177bb.png)
1. Reload apache to effect changes using ``sudo systemctl reload apache2``
1. Create a index.html file to serve from the projectlamp directory
![index file](https://user-images.githubusercontent.com/18899718/113885883-d383b200-9785-11eb-9a21-0b8639045569.png)
1. Check the website on the browser
![web](https://user-images.githubusercontent.com/18899718/113886089-03cb5080-9786-11eb-9ede-c1bc39762cf4.png)

# Enable PHP on the Website

1. Since our website is serving and index.html file, given that Apache's directoryIndex gives precedence to index.html over index.php, we need to change this order so we can check that our php is running smoothly.
We can do this by running ``sudo vim /etc/apache2/mods-enabled/dir.conf`` and reorder the files to bring index.php forward.
1. Save a close the file using and restart apache using ``sudo systemctl reload apache2``
1. We create a new php file in projectlamp using ``vim /var/www/projectlamp/index.php``
1. This will open a blank file, and write
```
<?php
phpinfo();
```
save and exit. Refresh web browser and we should have something like this 
![PHP info](https://user-images.githubusercontent.com/18899718/113888794-455cfb00-9788-11eb-9479-c5dc51bb0c0c.png)
5. For security reasons, we must remove the index page as it contains sensitve info. This can be done using ``sudo rm /var/www/projectlamp/index.php`` 