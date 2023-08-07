Here is a detailed documentation covering the entire process of setting up automated WordPress deployment on AWS using GitHub Actions as per the assignment requirements:

# Automated WordPress Deployment on AWS

## Introduction

The goal is to install WordPress on an EC2 instance in AWS and automate deployments using GitHub Actions as per the technical assignment requirements.

## Server Setup

- Launched an Ubuntu 22.04 EC2 instance on AWS.

- Configured security group to allow HTTP, HTTPS and SSH traffic.

- Updated packages:

---
- sudo apt update
- sudo apt upgrade
---

## Installed Nginx and optimized configuration:

---
- sudo apt install nginx -y
- sudo systemctl enable nginx
----
## Enable gzip compression and caching in /etc/nginx/nginx.conf

- Installed MySQL and created DB:

---
- sudo apt install mysql-server mysql-client -y
- sudo systemctl enable mysql
---
## To configure the database
---
- sudo mysql_secure_installation
  
## We need to set strong password, remove anonymous users, disallow root login, and delete the test database
## We configured database server
---
---
- mysql -u root -p 
- CREATE DATABASE wordpress;           # Created a new database
- CREATE USER 'user'@'localhost' IDENTIFIED BY "password";
- GRANT ALL PRIVILEGES ON "wordpress".* to "user";
- FLUSH PRIVILEGES;
- exit
---

## Installed PHP modules and configured php-fpm pool for Nginx.
---
- sudo add-apt-repository ppa:ondrej/php             # adding php packages to ubuntu repository
- sudo apt update 
- sudo apt install php7.4 php7.4-fpm php7.4-mysql php7.4-curl php7.4-gd php7.4-zip

## WordPress Installation

- Downloaded and extracted latest WordPress version.
  ---
 - sudo cp -r * /var/www/html                                # copied all wordpress files to /var/www/html
 - sudo chown -R www-data:www-data /var/www/html/            # giving permissions
 --- 

 ## Configuring nginx for wordpress
 ---
 - cd /etc/nginx/sites-enabled/
 - server_name mydomain.com # under server block if we have domain we need to configure here
 - we need to add index.php to list as server to know which file to open as wordpress written in php 
 - In location block we need to set /index.php?$args; this is basically configuration required to run wordpress site
 - Uncomment lines 'location' block as we installed php-fpm
 ---
 ## NOW WE CAN ACCESS WORDPRESS SITE FROM SERVER IP
 ## Configure database name, username, password, database host, table prefix
 ## Next site title, username, password, email
  
- Strong 15+ character passwords
- Limited admin user privileges 
- Disabled file editing from dashboard


## We need to  generate a self-signed SSL certificate using openssl as we dont have a domain name.
---
- sudo openssl genrsa -out ssl.key 2048
- sudo openssl req -new -key ssl.key -out ssl.csr
- sudo openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
- sudo cp ssl.crt ssl.key /etc/nginx/certs/
- sudo nano /etc/nginx/sites-available/default or wordpress
- ssl_certificate /etc/nginx/certs/ssl.crt;              # add these line under server block
- ssl_certificate_key /etc/nginx/certs/ssl.key; 
- sudo nginx -t
- sudo systemctl restart nginx
---


## Git Repository

- Initialized local git repository in web root and added WordPress files.

- Created GitHub repo and pushed code to master branch.

## GitHub Actions Workflow 

- Created .github/workflows/deploy.yml file in repo.

- Workflow checks out code, installs dependencies, deploys page via SSH.

- Leveraged secrets for private keys and IP addresses.

- Triggered on push requests to master branch.

- Here i used basic index.html page

## AWS additional security

- In addition to this i created VPC, in that i created two public subnets and two private subnets in different availability zones
  
- In public subnet i launched bastion host and in private subnets without public ip created two EC2 instances

- Created NAT gateway to allow private instance to download packages from internet

- Created and Attached Load balancer to public subnet to route traffic to both private instances using target groups
 
- Connected private instance in private subnet using bastion host and installed nginx, php and wordpress and configured wordpress server as above mentioned steps


## Testing 

- Verified automated deployments work as expected on push.

- Checked Nginx logs for errors. 

- Performed security audits.
