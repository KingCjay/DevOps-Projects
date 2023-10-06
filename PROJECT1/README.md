# LAMP STACK IMPLEMENTATION

## Linux Apache Mysql Php; where Linux is the operating system, Apache is the web server,Mysql is the database, and Php is the programming language.

## Prerequisites 

- Cloud Service Providers; AWS,Azure,GCP,etc.
- Launch a Linux instance on ubuntu(prefarbly).
- Priot knowledge on how to SSH into virtual host.

## Installation of a web server 

### Step 1 - Installing Apache abd Updating Firewall

Update server (an obligatory/best practice)

`sudo apt update`

Install Apache 

`sudo apt install apache2 -y` then -y for the YES prompt question to continue.

To verify if spache2 is running as a Service in your OS, used the following command:

`sudo systemctl status apache2`

![Alt text](images/apache-active.png)

It shows green and actively enabled. We can try to access it locally and as well as the internet. 

First let's check how we can access it locally from our shell:

`curl http://localhost:80` or `curl http://127.0.0.1:80`

Now, to test how our Apache HTTP server can respond to request from the internet. Open a web browser of your choice and try to access the following url;

`http://<Public-IP-Address>:80`

![Alt text](images/apache-works.png)


## Installation of database 

### Step 2 - Installing MySQL

Now that you have your web server upmand running, you need to install a __Database Management System(DBMS)__ to be able to manage and store data for your site in a relation database. MySQL is a popular relation database management system used within PHP enviroments, so we will use it in our project.

`sudo apt install mysql-server -y`

To check if MySQL is up and running, run the command:

`systemctl status mysql` 

![Alt text](images/sudo-mysql1.png)

Log in to your database console by typing;

`sudo mysql`

![Alt text](images/sudo-mysql.png)

It's recommended you run a security script the comes pre-installed with MySQL. This script with remove some default settings and lock down access to your database syetem. Before running the script, you'll set a password for the root user, using mysql_native_password as a default authentication 
method. We're definning users password as **PassWord.1**.

Run the command below;

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password PassWord.1`


Exit the MySQL shell with:

`mysql> exit`

Start the interactive script by running:

`sudo mysql_secure_installation`

This will ask if you want to configure the **VALIDATE PASSWORD PLUGIN**

**NOTE**: Enabling this feature is something of a judgement call. If enabled, passwords which don't match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique password for database credentials.

Answer **Y** for yes, or anything else to continue without enabling.

Copy Below Code
```php 
VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No:
```
If you answer "yes", you'll be asked to select a level of passsword validation. Keep in mind of the levels of password policies below.

```php
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
```
Regardless of whethr you chose to set up the **VALIDATE PASSWORD PLUGIN**, your server will next ask you to select and confirm a password for the MySQL root user. This is not to be confused with **system root**. The database root user is an administrative user with full privileges over the databse system.

Test if you're able to log in the MySQL system console by typing;

`sudo mysql -p`

![Alt text](images/mysql-p.png)

Notice the `-p` flag in this command, which will prompt for password used after changing the root user password.

Exit MySQL console by typing:

`mysql> exit`


## Installation of the Programming Language 

### Step 3 - Installing PHP

You have installed Apache to serve your content and MySQL to store and manage your data. PHP is the component setup that bwill process code to display dynamic content to end user. In addition to `php` package, you'll need `php-mysql` and `libapache2-mod-php`

`sudo apt install php` 

Also add/install some extra trivial commands for enabling your server 

`sudo apt install libapache2-mod-php php-opcache php-cli php-gd php-curl php-mysql`

Run the following command to see the version of PHP:

`php -v`

![Alt text](images/php-v.png)

### Restart the server 

`sudo systemctl restart apache2`

### Check if your LAMP is up

Go to your browser and type; 
localhost then go or
127.0.0.1
At this point, your LAMP stack is completely installed and fully operational.

To test your setup with a PHP script, it's best to set up a proper **Apache Virtual Host** to hold your website's folders and files. Virtual host allows you to have multiple websites located on a single machine and users of the websites will not even notice.

We will configure our first Virtual Host in the next step.

## Enable PHP on the website

### Step 5 - Enable PHP on the website

In the default **DirectoryIndex** settings in the Apache, a file named `index.html` will take precednce over an `index.php`file. This is useful 
setting up maintainance pages in PHP applications by creating a temporary `index.html`file containing an informative message for visitors. Because this page will take precedence over the index.php page, it will then become the landing pafe for the application. Once maintainance is over, the `index.html` is renamed or removed from the document root bringing back the regular application page.

Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server. This script confirms that Apache is able to handle and process requests for PHP files.

Create a new file named `index.php` inside your custom web root folder.

`vim /var/www/mylamp/index.php`

Enter the following PHP code inside the file:

```php
<?php
phpinfo();
```
When you are finished, save and close the file, refresh the page and you, get the image below.

![Alt text](images/php-v.png)

After checking the relevant information about your PHP server through that page, it's best to remove the file you created as it contains sensitive information about your PHP enviroment and your Ubuntu server. Run the code:

`sudo rm /var/www/mylamp/index.php`

you can always recreate this page if you want to access the information again later.

## Creating a Virtual Host for your Website using Apache

### Step 4 - Creating a Virtual Host for your Wesite using Apache

In this project, we set up a domain called `projectlamp` 

Apache on Ubuntu 22.0.4 has one server block enabled by default that is configured to serve documents from the /var/www/html directory. We'd leave the configuration as it is and add our own directory next to the default one.

Create directory for the `mylamp`

`sudo mkdir /var/www/mylamp`

Next, we assign ownership of the directory with the `$USER` enviroment variable, which will reference with your current system user:

`sudo chown -R $USER:$USER /var/www/mylamp`

Then, create and open a new configuration file in Apache's `sites-available` directory using your preferred command line editor.

`sudo vi /etc/nginx/site-available/mylamp.conf`

Copy and paste the configuration below inside the 

```php
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save and use `ls` to vie the content of the file:

`sudo ls /etc/nginx/sites-available/mylamp.conf`

![Alt text](images/sudo-ls-sites.png)

With this VirtualHost configuration, we are telling Apache to serve `mylamp` using `/var/www/mylamp`as its web root directory.

Now we use the `a2ensite` command to enable the new virtual host:

`sudo a2ensite mylamp`

We also need to disable the Apache's default website for it not to overwrite your virtual host. We do that by running the `a2dissite` command:

`sudo a2dissite 000-default`

To make sure your configuration doesn't contain syntax error, run:

`sudo apachectl configtest`

![Alt text](images/a2ensite.png)

Finally, reload Apache so the changes will take effect:

`sudo systemctl reload apache2`

Your new website is now active but the web root `/var/www/mylamp` is empty. Create an index.html file in that location so we can test if the virtual host is working.

`sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/mylamp/index.html`

Now go tour browser and to open your website url using IP address:

`http://<Public-IP-Address>:80`

![Alt text](images/IPworks.png)

If you see the `echo` command you wrote in the ***index.html*** file, then it means your Apache virtual host is working as expected. In the output, you'll see your server's public hostname(DNS Name) and public IP address. 
You can also access your website in your browser with the public (DNS Name) not only by public IP.  

`http://<Public-DNS-Name>:80`

![Alt text](images/DNSworks.png)

You can leave this file in place as temporary landing page for your applicationn until you setup an `index.php` file to replace it. Once you do, remember to remove or rename the `index.html`file from your document root as it would take precedence over an `index.php` file by default.

