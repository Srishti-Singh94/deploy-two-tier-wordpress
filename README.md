# deploy-two-tier-wordpress
Deploying a Two-Tier WordPress Application on Ubuntu 22 using AWS EC2 and RDS
This guide will walk you through the steps to deploy a two-tier WordPress application on Ubuntu 22 using AWS EC2 for the web server and AWS RDS for the database.

Prerequisites
An AWS account with appropriate permissions.
Basic familiarity with AWS services.
A domain pointing to the ec2's ip
Step 1: Launch the EC2 Instance
Log in to your AWS Management Console.
Navigate to EC2 Dashboard.
Click "Launch Instance" and choose an Ubuntu 22 AMI.
Select an instance type suitable for your workload.
Configure instance details (VPC, subnet, security group, etc.).
Add storage as needed.
Configure the security group to allow HTTP (port 80), HTTPS (port 443) and SSH (port 22) traffic.
Review and launch the instance, selecting your SSH key pair.
Connect to the instance using SSH: ssh -i /path/to/your/key.pem ubuntu@your-instance-ip.
Step 2: Launch the RDS Instance
Navigate to the RDS Dashboard in AWS.
Click "Create database" and choose MySQL.
Configure DB settings (instance type, storage, username, password, etc.).
Configure advanced settings, including VPC, subnet, and security group.
Allow the rds security group to receive inbound request for mysql from ec2 instance's security group.
Create the database instance.
Copy the credentials for further use.
Step 3: Set Up LAMP Stack (EC2)
Update and upgrade packages: sudo apt update && sudo apt upgrade -y.
Install Apache: sudo apt install apache2 -y.
Install php and required packages: sudo apt install -y php libapache2-mod-php php-mysql  php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip.
Create Virtual Host for website:
sudo mkdir /var/www/your_domain
sudo chown -R $USER:$USER /var/www/your_domain
Create the configuration file for your domain sudo nano /etc/apache2/sites-available/your_domain.conf.
Paste this configuration, and modify the required values
<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /var/www/your_domain>
      AllowOverride All
    </Directory>
</VirtualHost>
Press Ctrl+O and Enter to save, and Ctrl+X to exit.
Enable our site using sudo a2ensite your_domain.
(optional) disable the default apache site using sudo a2dissite 000-default.
Enable the rewrite module to be able to use Wordpress permalink feature sudo a2enmod rewrite.
Make sure the configuration is correct sudo apache2ctl configtest.
Reload apache sudo systemctl reload apache2.
Step 4: Check MySQL connection and create required database
Install MySQL Client: sudo apt install mysql-client -y.
Login to mysql using credentials you got from RDS mysql -h <hostname-from-rds> -u <rds-username> -p
Enter the password. Check if the connection is successful.
Create database create database wordpress;
Exit mysql using exit.
Step 5: Download Wordpress
Move to tmp directory cd /tmp.
Download the latest version of wordpress curl -O https://wordpress.org/latest.tar.gz.
Extract the compressed file tar xzvf latest.tar.gz.
Create an empty .htaccess file to be used by wordpress and apache touch /tmp/wordpress/.htaccess.
Create the upgrade directory so that wordpress can easily update on it's own mkdir /tmp/wordpress/wp-content/upgrade.
Copy the content of wordpress to your document root sudo cp -a /tmp/wordpress/. /var/www/your_domain.
Adjust the ownership of the directory and their folders and files
sudo chown -R www-data:www-data /var/www/your_domain
sudo find /var/www/your_domain/ -type d -exec chmod 750 {} \;
sudo find /var/www/your_domain/ -type f -exec chmod 640 {} \;
Now your website should be available on specified domain
Step 6: Setup SSL Certificate for HTTPS
Install Certbot sudo apt install certbot python3-certbot-apache.
Run certbot and provide all the required options sudo certbot --apache.
Congratulations! You've successfully deployed a two-tier WordPress application on Ubuntu 22 using AWS EC2 and RDS.
