# Automating Loadbalancer configuration with Shell Scripting

Here we demonstrate how to automate setup and maintanance of your loadbalancer using a freestyle job, enhancing efficiency and reducing manual effort.

## Automate the Deployment of Webservers 

In ths project, we will be writing a shell script that when ran, all that we did manually in the course **Implementing loadbalancer with Nginx** , will be done automatically. Automation helps us speed the deployment of services and reduce the chance of making errors in our in day to day activities.

## Deploying and Configuring the Webservers 

All the process we need to deploy our webservers has been confided in the shell script below:

Copy below code
```php
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```

Follw the steps below to run the script:

**Step 1**: Provision an EC2 instance running ubuntu 22.04.3. in the two webservers.

**Step 2**: Open port 8000 to allow traffic from anywhere using the security group

**Step 3**: Connect to the web server via the terminal using SSH client

**Step 4**: Open a file and paste the script above 

**Step 5**: Change permissions on the file to make it executable with the command 
`sudo chmod u+x install.sh`

**Step 6**: Run the script with the command below

`./install.sh PUBLIC_IP_ADDRESS`

## Deployment of Nginx as a Loadbalancer using the Shell Script

### Automate the Deployment of Nginx as a Loadbalancer using the Shell Script

Having sucessfully deployed and configured two webservers, We will move on to the loadbalancer. As a prerequiste, we need to provisoin an EC2 instance running on ubuntu 22.04.3, open port 80 to anywhere using the security group and connect to the load balancer via the terminal.

## Deploying and Configuring Nginx LoadBalancer 

All the steps followed  in the **Implementing loadbalancer with Nginx** course has been codified below: 

```php
#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx
```

### Steps to run the shell script

**Step 1**: On your terminal, open a file nginx.sh

**Step 2**: Copy and paste the above script inside the file

**Step 3**: Change the file permission to make it an executable using the command below 

`chmod u+x nginx.sh`

**Step 4**: Run the script with the command 

`./nginx.sh PUBLIC_IP_ADDRESS Webserver_1 Webserver_2`

In my project, its like this ***./nginx.sh 16.171.15.215 16.170.254.211:8000 13.53.40.240:8000*** 

**Result**: The Nginx Server acts as a loadbalancer for the two websevers. The webservers changes as you reload your nginx server.

![Alt text](<images/Screenshot 2023-10-14 at 17.18.49.png>)

![Alt text](<images/Screenshot 2023-10-14 at 17.19.06.png>)

![Alt text](images/auto-server1.png)

![Alt text](images/auto-server2.png)