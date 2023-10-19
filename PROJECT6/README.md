# Implementing Wordpress website with LVM Storage management

In this course, we're to learn how to build and manage a scalable Wordpress Website using AWS EC2 and LVM(Logical Volume Management) storage. We'll gain proficiency in installing and configuring WordPress, customizing themes, and adding essential plugins to enhance our websites's functionality.  

## Prerequisites

- Cloud Service Providers; AWS,Azure,GCP,etc.
- Launch a Linux instance on RedHat.
- Priot knowledge on how to SSH into virtual host.

## Understanding 3 Tier Architecture 

### Web Solution With WordPress

In this project, we will be preparing storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and opn-source content management system written in **PHP** and paired with **MySQL** or **MariaDB** as its backend Relation Database Management System(RDBMS).

This Project consists of two parts: 

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of the part is to give you practical experience of working with disks, partition and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skill of deploying Web and DB tiers of Web solution.

#### Three-tier Architecture 

Generally, web or moblie solutions are implemented based on what we call the **Three-tier Architecture**.

Three-tier architecture is a well-established software application architecture that organizes applications into three logical and physical computing tiers: the presentation tier, or user interface; the application tier, where data is processed; and the data tier, where the data associated with the application is. 

3-Tier Setup

1. A laptop or PC to serve a a client.
2. An EC2 Linux Server as a web server(This is where we will install Wordpress)
3. An EC2 Linux server as a database (DB) server

***Use `RedHat` OS for this project***

**Note**: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we use `ubuntu` user, but for RedHat you will need to use `ec2-user` user. Connection string will look like `ec2-user@<Public-IP>`. 

## Implementing LVM on Linux servers(Web and Database servers)

#### Prerequisites 



**Step 1**: Prepare Web Server

1. In your instance that will serve as "WebServer", create # volumes in the same AZ as your Web Server EC2, each of 10GIB.

![Alt text](images/Create-Volumes.png)

2. Attach all three volumes one by one to your Web Server EC2 instance.

![Alt text](<images/attach-vol 2.png>)

Open the Linux terminal to begin configuration

3. Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/directory. Inspect with `ls /dev/` and make sure you see all 3 newly created block devices there- their names likely to be `xvdf`,`xvdh`,`xvdg`.

![Alt text](<images/lsblk1 2.png>)

4. Use `df-h` command to see all mounts and free space your server.

5. Use `gdisk` utility to create a single partition on each of the 3 disks.

`sudo gdisk /dev/xvdf`

![Alt text](images/GPT-partition.png)

Use `lsblk` utility to view the newly configured partition on each of the disks.

![Alt text](images/lsblk-3disks.png)

6. Install `lvm2` package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

![Alt text](<images/lvmdiskscan 2.png>)

7. Use `pvcreate` utility to mark each of the 3 disks as physical volumes (PVs)to be used by LVM.

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

8. Verify that your Physical volume has been created successfully by running 

`sudo pvs`

![Alt text](images/sudo-pvs.png)

9. Use `vgcreate` utility to add all 3 PVs to a volume group(VG). Name the VG **webdata-vg**.

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

10. Verify that your VG has been created successfully by running 

`sudo vgs`

![Alt text](images/sudo-vgs.png)

11. Use `lvcreate` utility to create 2 logical volumes.**apps-lv (***Use half of the PV size***)**, and **logs-lv (***Use the remaining space of the PV size.***)**NOTE**: apps-lv will be used to store data for the Website while, logs-lv will be used to stored data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

12. Verify that your Logical Volume has been created successsfully by running 
`sudo lvs`

![Alt text](images/sudo-lvs.png)

13. Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`
`sudo lsblk`

![Alt text](images/sudo-lsblk.png)

14. Use `mkfs.ext4` to format the logical volumes with `ext4` filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

15. Create **/var/www/html** directory to store website files

`sudo mkdir -p /var/www/html`

16. Create **/home/recovery/logs** to store backup of log data

`sudo mkdir -p /home/recovery/logs`

17. Mount **/var/www/html** on **apps-lv** logical volumes

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

18. Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (***This is required before mounting the file system***)

`sudo rsync -av /var/log/. /home/recovery/logs `

19. Mount **/var/log** on **logs-lv** logical volume. (***Note that all existing data on /var/log will be deleted. That is why the step 15 above is very important***)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

20. Restore log files back into **/var/log** directory

`sudo rsync -av /home/recovery/logs/. /var/log`

21. Update `/etc/fstab` files so that the mount configuration will persist after restart of the server.

The UUID of the device will be used to update the `/etc/fstab` file;

`sudo blkid`

![Alt text](images/sudo-blkid.png)

`sudo vi /etc/fstab`

Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending qoutes.

![Alt text](images/cat-etc-fstab.png)

22. Test the configuration and reload the daemon

`sudo mount -a`
`sudo systemctl daemon-reload`

23. Verify your setup by running `df -h`, output must look like this:

![Alt text](images/df-h-23.png)

# Installing wordpress and configuring to use Mysql Database

**Step 2** : Prepare Database Server

We launch a second RedHat EC2 instance with the role “DB server,” and we repeat the same steps as for the Web Server, but instead of `apps-lv`, we created` db-lv` and mounted it to `/db` directory instead of  `/var/www/html/`.

**Step 3** : Install Wordpress on Web Server EC2

1. Update the repository

`sudo ym update`

2. Install wget, Apache and its dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

3. Start Apache

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

4. To install PHP and its dependencies

Copy below;

```php
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

6. Download wordpress and copy wordpress to `/var/www/html`

Copy Below Code
```php
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```

7. Configure SELinux Policies 

Copy Below Code
```php
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

**Step 4**: Install MySQL on your DB Server EC2

`sudo yum update`

`sudo yum install mysql-server`

Verify the service is up and running, if it's not up and running, restart the service and enable it so it will be running even after reboot.

`sudo systemctl status mysqld` 

`sudo systemctl restart httpd`

`sudo systemctl enable mysqld`

**Step 5** : Configure DB to work with Wordpress

`sudo mysql`
```php

mysql> CREATE DATABASE Wordpress;
       CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
       GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
       FLUSH PRIVILEGES;
       SHOW DATABASES;
exit
```

![Alt text](images/mysql-wordpress.png)

**Step 6**: Configure Wordpress to connect to remote database

We open MySQL port 3306 on DB Server EC2. For extra security, we allow access to the DB server ONLY from our Web Server’s IP address, so in the Inbound Rule configuration, we specify the source as /32

![Alt text](images/db-server.png)

1. Install MySQL Client and test if we can connect successfully to Wordpress DB on the DB server.

`sudo yum install mysql`

`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`



2. Verify if you can successfully `SHOW DATABASES;`

![Alt text](images/mysql-webserver.png)

3. Change permissions and configurations so Apache could use Wordpress:

`cd /var/www/html/wordpress`

`sudo vi wp-config.php`

![Alt text](images/cat-wp-configphp.png)

We edit the wp-config.php file and fill it with our DB details.

4. We enable TCP port 80 in inbound rules configuration for our Web Server EC2 (enable from everywhere 0.0.0.0/0 or our workstation’s IP)


5. We can now access our WordPress from our browser with the link;

`http://<Web-Server-Public-IP-Address>/wordpress/`

![Alt text](images/start-wordpress-installation.png)

![Alt text](images/welcome-to-wordpress.png)