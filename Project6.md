# Web Solution With WordPress

## Project goals:
1. Configure storage subsystem for Web and Database servers based on Linux OS. This focuses on working with disks, partitions and volumes in Linux.
1. Install WordPress and connect it to a remote MySQL database server

## Setup Webserver [webserver](#webserver)
1. Lunch 2 redhat EC2 instances in AWS. Label one `webserver` and the other `database server`.
1. We create and attach 3 10gb volumes to our `webserver` making sure the volumes are in the same `availability zone` as our `webserver`
1. Login to the `webserver` and run `lsblk` command to see our newly attached devices. listed as `xvdf`, `xvdg` and `xvdh` respectively.
![](https://user-images.githubusercontent.com/18899718/117910151-0814ed00-b2a1-11eb-966e-03f6ba12fe2d.png) 
1. Next, we create a single partition on each of our newly created 3 disks.
1. Run `sudo gdisk /dev/xvdf`
1. The `gdisk` utility will give us screen prompts where we input letters according to what kind of logical volume we're creating. First we choose `n` to indicate a `new partition`, For this partion, we're using the entire disk, so for `partion number` we type in `1` which is the default number. For `first sector` and `last sector`, we hit enter to leave them at their default values.
1. Next we input the HEX code for the type of partition we want, since we're making a `Logical Volume Management(LVM)` disk, we type in `8e00` which will change it to `Linux LVM`
1. Type `p`to display partion summary data.
1. Then type `w` to write data, this saves the changes we've made. 
1. A screen prompt would ask if you want to proceed, type `yes` and the partition would be created for `/dev/xvdf`
1. Repeat these steps 5-10 for `/dev/xvdg` and `/dev/xvdh`
1. Run `lsblk` again and you would notice each of the newly partioned devices would have an attached devices reading `xvdf1`, `xvdg1` and `xvdh1`
1. Next, we install `lvm2` using `sudo yum install lvm2`
1. We create physical volumes using
`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`. 
1. Run `sudo pvs` ![](https://user-images.githubusercontent.com/18899718/117913076-4f51ac80-b2a6-11eb-919c-ee08be37b525.png)
1. Next, we create a `volume group` named `webdata-vg` by running 
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
17. Verify volume group is successfully created by running `sudo vgs` ![](https://user-images.githubusercontent.com/18899718/117913334-d010a880-b2a6-11eb-994e-5a94d02abf1c.png)
1. We use `lvcreate` utility to create 2 logical volumes namely `apps-lv`(which would be used to store data for the website) and `logs-lv` (this will be used to store data for logs). We divide our physical volume between them withe each getting equal halves. 
``` 
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
19. Run `sudo lvs` to verify that the Logical Volume has been successfully created.

### format logical Volumes
1. We use `mkfs.ext4` to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![](https://user-images.githubusercontent.com/18899718/117916718-4fa17600-b2ad-11eb-8027-5ac71b1fef6b.png)

### Create Directories and Mount Logical Volumes
1. Create /var/www/html directory to store website files
`sudo mkdir -p /var/www/html`
1. Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`
1. Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`
1. Backup all files in log directory `/var/log` into` /home/recovery/logs` using `rsync` utility.
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
5. Mount `/var/log` on `logs-lv` logical volume
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
6. Restore log files back into `/var/log` 
directory
```
sudo rsync -av /home/recovery/logs/log/. /var/log
```
7. We make the our mount persist by updating `/etc/fstab/` with our LV `UUID` 
1. Run `sudo blkid` to get `UUID`number
1. Run `sudo vi /etc/fstab` and update as shown in the image below ![](https://user-images.githubusercontent.com/18899718/117915780-97270280-b2ab-11eb-9a08-92d5b1370c78.png)
1. Test configureatio and reload deamon
```
sudo mount -a
sudo systemctl daemon-reload
```
![](https://user-images.githubusercontent.com/18899718/117915957-e2d9ac00-b2ab-11eb-8b90-12af36760aa7.png)



## Configure Database Server [server]()
1. Follow step 1-15 in **Setup Webserve** above.
2. Next, we create a `volume group` named `database-vg` by running 
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
![](https://user-images.githubusercontent.com/18899718/117916908-b030b300-b2ad-11eb-9f87-5717038b60aa.png)
3. We use `lvcreate` utility to create 1 logical volume namely `db-lv`
![](https://user-images.githubusercontent.com/18899718/117917266-57ade580-b2ae-11eb-8474-63cee6fc3161.png)

4. create directory `db` to mount our logical volume.
`sudo mkdir /db`
5. We use `mkfs.ext4` to format the logical volume with ext4 filesystem
```
sudo mkfs -t ext4 /dev/database-vg/db-lv
```
![](https://user-images.githubusercontent.com/18899718/117917612-1538d880-b2af-11eb-97d6-4aeda2a53cbf.png)

6. We mount our logical volume
`sudo mount /dev/database-vg/db-lv /db`

7. We make the our mount persist by updating `/etc/fstab/` with our LV `UUID` 
8. Run `sudo blkid` to get `UUID`number
9. Run `sudo vi /etc/fstab` and update as shown in the image below ![](https://user-images.githubusercontent.com/18899718/117917936-bf186500-b2af-11eb-81dc-8f337b9acb83.png)
1. Test configureatio and reload deamon
```
sudo mount -a
sudo systemctl daemon-reload
```

## Install wordpress on webserver
1. Update repository
`sudo yum -y update`
1. Install wget, Apache and it’s dependencies
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
1. Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
``` 
![](https://user-images.githubusercontent.com/18899718/117919221-39e27f80-b2b2-11eb-8c8b-d11960466d9d.png
)
4. install PHP and it’s dependencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
5. Restart Apache
`sudo systemctl restart httpd`

1. Download and install wordpress
create a directory and download wordpress
```
mkdir wordpress & cd wordpress
sudo wget http://wordpress.org/latest.tar.gz
```
7. Extract wordpress `sudo tar xzvf latest.tar.gz`
1. we change directory to the new wordpress directory
`cd wordpress`
1. creat and copy config file
```
sudo cp wp-config-smaple.php wp-config.php
```

10. Copy the content of wordpress folder to `/var/www/html`. First, change direcotry to the wordpress folder we created
```
cd ..
cp -R wordpress/. /var/www/html/
```
11. install mysql client on webserver and database server
`sudo yum -y install mysql-server`
1. on webserver, run 
```
sudo systemctl enable mysqld
sudo systemctl start mysqld
```
12. on database server, run
```
sudo systemctl enable mysqld
sudo systemctl start mysqld
```
13. configure mysql
`sudo mysql_secure_installation`
follow the screen prompt to complete installation
14. We create database user
```
CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpress';
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
15. Create database
```
CREATE DATABASE wordpress;
```
16. Exit
1. Set bind address so our webserver would be able to connect.
```
sudo vi /etc/my.cnf
```
![](https://user-images.githubusercontent.com/18899718/117922317-dd825e80-b2b7-11eb-8fe0-d0a536a52092.png)

18. Restart mysql service
```
sudo systemctl restart mysqld
```
19. Back to webserver, edit config.php 
in /var/www/htm/worpress folder, run
```
sudo vi config.php
```
20. update the file with our database server details
![](https://user-images.githubusercontent.com/18899718/117922912-d019a400-b2b8-11eb-9e19-a3d6c21580fb.png)
20. Save and exit. 
21. Restart Apache service
```
sudo systemctl restart httpd
```
22. Disable Apache default page
```
mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.backup
```
23. In security group on our AWS database instance, open port 3306 for MYSQL/Aurora so our webserver would be able to communicate
24. Let's see if our webserver would communicate with our database by running 
the command in the image below ![](https://user-images.githubusercontent.com/18899718/117923775-42d74f00-b2ba-11eb-9edf-e676e9f6c8fe.png)
25. Lastly, we configure SElinux policies on our webserve
```
sudo chown -R apache:apache /var/www/html/

sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R

sudo setsebool -P 

httpd_can_network_connect=1
```
26. Now, we go to our webserver public ip adress and reload it. 
![](https://user-images.githubusercontent.com/18899718/117924445-6058e880-b2bb-11eb-9975-84395e544a5b.png)

27. After installation, we've had wordpress dashboard that looks like this
![](https://user-images.githubusercontent.com/18899718/117924681-b594fa00-b2bb-11eb-9d4a-b61f5b7a7b35.png)