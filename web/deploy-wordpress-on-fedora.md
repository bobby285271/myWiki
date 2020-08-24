---
title: How to Deploy WordPress on a Fedora Server
description: 
published: true
date: 2020-08-24T10:17:41.282Z
tags: web, guide, linux, fedora, wordpress, deploy
editor: markdown
---

First things first
------------------

You should get a IP address after buying VPS, such as `123.123.123.123`. Open your favourite terminal and enter:

> Replace `123.123.123.123` with your actual IP address.

```
ssh root@123.123.123
```

Enter your password to login. Perform a full upgrade on your system:

    dnf upgrade

Package Installation
--------------------

Install Apache, PHP and MariaDB using the DNF package manager. There is NO need to install the `wordpress` package.

    dnf install @"Web Server" php-mysqlnd mariadb-server

Enable the web and database services to start at boot time, then start them immediately:

```
systemctl enable httpd.service mariadb.service
systemctl start httpd.service mariadb.service
```

MariaDB Configuration
---------------------

Firstly, initialize MariaDB. If this is your first use of MariaDB, you should create a password for your root user here. **Don’t use** the system’s own root (administrator) password. It is suggested to answer `y` to all yes-no questions afterwards:

```
mysql_secure_installation
```

Next, create a database. You can host more than one WordPress site on a machine. Therefore, you may want to choose a distinctive name for yours. For instance, this example uses `mywpsite`. The `-p` switch prompts you for a password.

> Replace `mywpsite` with a database name you preferred.

```
mysqladmin create mywpsite -u root -p
```

Next, set up a special privileged user and password for the database. The web app uses these credentials to run. Use the standard `mysql` client program for this step. The `-D` option attaches to the built-in MySQL database where privileges are stored.

> Replace `sqluser` and `password` with a user name you preferred and a strong password.

```
mysql -D mysql -u root -p
```
```
GRANT ALL PRIVILEGES ON mywpsite.* TO 'sqluser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
QUIT;
```

Set up the Web Server
---------------------

Next, tune the SELinux parameters so the web server can perform necessary functions.

```
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_can_sendmail=1
```

Next, configure your firewall so it allows traffic on port `80` (HTTP):

```
firewall-cmd --add-service=http --permanent
firewall-cmd --reload
```

Download WordPress
------------------

These latest version of WordPress are always available on `https://wordpress.org/latest.tar.gz`.

```
cd /var/www/html
wget https://wordpress.org/latest.tar.gz
```

Extract the downloaded archive to the document root of your domain and update permissions on files.

```
tar xzf latest.tar.gz
chown -R apache.apache wordpress
chmod -R 755 wordpress
```

All Done
--------

Visit `123.123.123.123/wordpress`, and finish the installation.