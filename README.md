# Linux Server Configuration

## Overview

This is the sixth project for the Udacity Full Stack Nanodegree. This project is about taking a baseline installation of a Linux server and prepare it to host my web applications.  We secured my server from a number of attack vectors, install and configure a database server, and deploy one of my existing web applications onto it. 

## Server Info

- **Public IP:** 52.4.173.75
- **Port:** 2200

## Launch Virtual Machine

Use [Amazon Lightsail](https://amazonlightsail.com/) to create a Linux server instance. 

#### Start a new Ubuntu Linux server instance on Amazon Lightsail.

- Sign up for an account if you do not have an account, or log in. 
- Create an instance.
- Pick your instance image --> Linux/Unix --> OS Only --> Ubuntu(16.04 LTS).
- Choose your instance plan (lowest tier is fine).
- Give your instance a hostname.
- Wait for startup, it takes several minutes. 
- Once the instance has started up, follow the instruction provided to SSH into your serve.  If you are a windows user, click [here](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh) for the instruction. But from my own experience, the Ubuntu system is better to use. 

#### Create a new user named grader and log in old port 22 first： 

- Create a new user account named grader.

  `sudo adduser grader`

- Give grader the permission to sudo.

  ```
       ssh-keygen -t rsa
        
    To login
        ssh -i .ssh/grader_udacity grader@52.4.173.75 -p 22
  ```

- Give grader the permission to sudo.

  ```
    sudo vim /etc/sudoers.d/grader
  ```

  add text `grader ALL=(ALL) NOPASSWD:ALL`

#### Secure your serve: 

- Update all currently installed packages. 

  ```
   sudo apt-get update
    sudo apt-get upgrade
  ```

- Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

  ```
    sudo vim /etc/ssh/sshd_config
  ```

  change port form 22 to 2200

- Configure the Uncomplicated Firewall (UFW) to only allow incoming  connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

  ```
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/tcp
    sudo ufw enable
  ```

  **Warning: ** When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app `Connect using SSH'`button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

#### Configure the local time zone to 

1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to `US/Eastern`. 

## Install Relative Sofware

#### Install and configure Apache to serve a Python application

- Install Apache `sudo apt-get install apache2`

- Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`

- Restart Apache `sudo service apache2 restart`

#### Install and configure PostgreSQL

- Install PostgreSQL `sudo apt-get install postgresql`

- Check if no remote connection are allowed `sudo vim /etc/postgresql/9/3/main/pg_hba.conf`

- Login as user "postgres" by `sudo su -postgres`

- Get into postgreSQL shell `psql`

- Create a new database named catalog  and create a new user named catalog in postgreSQL shell

  ```
  CREATE DATABASE catalog;
  CREATE USER catalog;
  ```

- Set a password for user catalog

  ```
  ALTER ROLE catalog WITH PASSWORD 'password';
  ```

- Give user "catalog" permission to "catalog" application database

```
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

- Quit postgreSQL by  `postgres=# \q`

- Exit from user "postgres" by `exit`

#### Install git, clone and setup your Catalog App project.

1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `https://github.com/Yanlingwu95/Item-Catalog.git`
6. Rename the project's name `sudo mv ./Item_Catalog ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `app.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `database_setup.py`, `app.py` and `database_init.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo pip install -r requirements.txt`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`

#### Configure and Enable a New Virtual Host

1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`

2. Add the following lines of code to the file to configure the virtual host.

   ```
   <VirtualHost *:2200>
   	ServerName 52.4.173.75
   	ServerAdmin yanlingwu95@gmail.com
   	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
   	<Directory /var/www/FlaskApp/FlaskApp/>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	Alias /static /var/www/FlaskApp/FlaskApp/static
   	<Directory /var/www/FlaskApp/FlaskApp/static/>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	ErrorLog ${APACHE_LOG_DIR}/error.log
   	LogLevel warn
   	CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

#### Create the .wsgi File

1. Create the .wsgi File under /var/www/FlaskApp:

   ```
   cd /var/www/FlaskApp
   sudo nano flaskapp.wsgi 
   ```

2. Add the following lines of code to the flaskapp.wsgi file:

   ```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/FlaskApp/")
   
   from FlaskApp import app as application
   application.secret_key = 'Add your secret key'
   ```

#### Restart Apache

- Restart Apache `sudo service apache2 restart `

## References:

-  <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

- <https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04>

- <https://help.ubuntu.com/community/PostgreSQL>

