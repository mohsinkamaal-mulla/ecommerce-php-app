# Introduction

This is a sample e-commerce application built for learning purposes.

Here's how to deploy it on CentOS systems:

## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum install -y firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl status firewalld
```

## Deploy and Configure Database

1. Install MariaDB

```
sudo yum install -y mariadb-server
sudo vi /etc/my.cnf
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```

2. Configure firewall for Database

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```

3. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

4. Load Product Inventory Information to database

Create the db-load-script.sql

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

Run sql script

```

sudo mysql < db-load-script.sql
```

## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysqlnd
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

4. Download code

```
sudo yum install -y git
sudo git clone https://github.com/mohsinkamaal-mulla/ecommerce-php-app.git /var/www/html/
```

5. Update index.php

Update [index.php](https://github.com/mohsinkamaal-mulla/ecommerce-php-app/edit/main/index.php#L107) file to connect to the right database server. In this case `localhost` since the database is on the same server.

```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php

```

> ON a multi-node setup remember to provide the IP address of the database server here.
```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php
```

6. Test

```
curl http://localhost
```

## Automated Deployment Script

Here you can find the deployment script for automation

https://github.com/mohsinkamaal-mulla/shell-scripting/blob/main/ecommerce_deployment_script.sh
