# LEMP STACK IMPLIMENTATION

## Install Nginx Server
1. Update package index using ``sudo apt update`` and install nginx using ``sudo apt install nginx``
1. Verify ngix has been sucessfully installed. 
![nginx server](https://user-images.githubusercontent.com/18899718/114393794-0d76fe80-9b60-11eb-9984-abc8f388499a.png)
1. Check to see we can access our server locally by running a ``curl` command
![curl](https://user-images.githubusercontent.com/18899718/114394121-75c5e000-9b60-11eb-9b8d-050981f28eba.png)
1. We check to see if our server would is accesible online via the public IP address. 
![Check nginx page](https://user-images.githubusercontent.com/18899718/114397357-3ac5ab80-9b64-11eb-857e-dab6d12af903.png)

## Install MySQL

1. We acquire and install the MySQL using ``sudo apt install mysql-server``
1. Once done, run ``sudo mysql_secure_installation`` The interactive script that removes inscure setting. 
1. Test if you can access MySQL console by typing ``sudo mysql`` 
![mysql](https://user-images.githubusercontent.com/18899718/114399163-3ac6ab00-9b66-11eb-9184-fa512447ec24.png)
1. To exit the console, we type ``exit``

## Install PHP
1. To get PHP working, we running the ``sudo apt install php-fpm php-mysql`` command.

## Congifure Nginx to use PHP

1. We create a root directory for our project -projectLEMP using ``sudo mkdir /var/www/projectLEMP``
1. We assign ownership of the directory with the $USER environment variable by typing ``sudo chown -R $USER:$USER /var/www/projectLEMP``
1. We create a new configuration file in Nginx’s ``sites-available`` directory by typing ``sudo nano /etc/nginx/sites-available/projectLEMP
``
1. This will create a new blank file. we copy the configuration below inside. 
![configuration](https://user-images.githubusercontent.com/18899718/114400604-bf65f900-9b67-11eb-95e7-e6082c7fa2ff.png)
1. Link the configuration file to the `sites-enabled` directory
``sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`` 
1. To make sure there a no errors, test configuration `` sudo nginx -t``
![ngix syntax](https://user-images.githubusercontent.com/18899718/114401285-634fa480-9b68-11eb-8d64-7026ecfddc1d.png)
1. We disable the defualt Nginx host which is configured to listen on port 80 ``sudo unlink /etc/nginx/sites-enabled/default``
1. We reload Nginx by typing ``sudo systemctl reload nginx``
1. We creat an index.html file in  /var/www/projectLEMP since it's still empty. ``sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html``
We validate this on our web browser.
![](https://user-images.githubusercontent.com/18899718/114402842-d9a0d680-9b69-11eb-95e6-1b22a5b6084a.png)

## Test PHP with Nginx

1. Now we make sure Nginx can correctly server files to PHP processor, we do this by a test document named ``info.php`` we type
``nano /var/www/projectLEMP/info.php``
1. We type 
```
?php
phpinfo();
```
3. We go to our browser to make sure we can access the page.
![](https://user-images.githubusercontent.com/18899718/114404127-0a354000-9b6b-11eb-85b6-0924c1e02a96.png)
1. Since this file contains sensitive information about our server, we remove it by typing ``sudo rm /var/www/your_domain/info.php``

## Retrieve data from MySQL database using PHP

1. We create a test “To do list” database and configure access to it, so our Nginx website would be able to query data from the DB and display it.
1. we loging to the mysql ``sudo mysql``
1. We create a new database ![](https://user-images.githubusercontent.com/18899718/114404897-c0992500-9b6b-11eb-9767-67f0901faf11.png)
1. We create a new user and password for the database ![](https://user-images.githubusercontent.com/18899718/114405119-f5a57780-9b6b-11eb-97fc-9f65fac2602f.png)
1. We give our new user permission over the database we created earlier
![](https://user-images.githubusercontent.com/18899718/114405518-6351a380-9b6c-11eb-99a4-2ee7fe98dcd1.png)
1. We exit mysql shell using ``exit``
To log into mysql with the new user and password, we type ``mysql -u example_user -p``
1. Type ``SHOW DATABASES;``
![](https://user-images.githubusercontent.com/18899718/114406366-19b58880-9b6d-11eb-85b9-6c93fbc698ee.png)
1. We then create a test table named todo_list ![](https://user-images.githubusercontent.com/18899718/114406632-4ff30800-9b6d-11eb-9875-4e2a259e4b56.png)
1. We insert a few rows in the table using different values. ![](https://user-images.githubusercontent.com/18899718/114407272-d7d91200-9b6d-11eb-9438-0fe4cb7b5b26.png)
1. Exit mysql
1. We create a PHP script that will connect to MySQL and query for our content. We create a new PHP file in your custom web root directory
``nano /var/www/projectLEMP/todo_list.php
``
1. The paste the following content in the new file. ![](https://user-images.githubusercontent.com/18899718/114410448-00164000-9b71-11eb-883e-924ca255b364.png). We save and close the file when done editing.
1. We access the page through our public ip address and add ``/todo_list.php`` to view our to-d0 list page.
![](https://user-images.githubusercontent.com/18899718/114411340-b4b06180-9b71-11eb-809d-6914aee879e5.png)