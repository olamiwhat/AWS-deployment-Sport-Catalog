# LINUX SERVER CONFIGURATION
This is the fifth and final project of the [Udacity's Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004) Program. This project involves taking a baseline installation of a Linux Server and preparing it to host web application (in this case, the Item Catalog Website), installing and configuring a database server on it aand securing it against a number of attack vectors.
## Project Requirement
### Get server
1. Start a new Ubuntu Linux server instance on [Amazon Lightsail.](https://aws.amazon.com/lightsail/)
2. SSH into server
### Secure server
3. Update all currently installed packages
4. Change the SSH port from 22 to 2200
5. Configure Uncomplicated Firewall(UFW) only allowing incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
### Give `grader` access
6. Create a new user account named `grader`.
7. Give `grader` the permission to `sudo`.
8. Create an SSH key pair for `grader` using the `ssh-keygen tool`.
### Prepare to Deploy Project
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install and configure PostgreSQL:
    - Do not allow remote connections
    - Create a new database user named `catalog` that has limited permissions to your catalog application database.
12. Install `git`.
### Deploy the Item Catalog Project
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your `.git` directory is not publicly accessible via a browser!
## Getting Started
### The Linux Server
- A Ubuntu Linux Server Instance on [Amazon Lightsail](https://aws.amazon.com/lightsail/). .
- Public Static IP Address: **35.170.214.60**
- Hostname: ec2-35-170-214-60.compute-1.amazonaws.com
### SSH into server
SSH into server by clicking on the **connect using ssh** button on the connect tab. This should open a terminal in the browser. Login using ubuntu(no password). Further instruction on how to ssh [here](https://lightsail.aws.amazon.com/ls/docs/en_us/articles/understanding-ssh-in-amazon-lightsail) 
### Secure Server
6. Update all currently installed packages - in terminal type:
     - `sudo apt-get update`
     - `sudo apt-get upgrade`
7. Change SSH port from 22 to 2200
    - Edit the sshd_config file by typing `sudo nano /etc/ssh/sshd_config` and then change port 22 to port 2200. save and quit
    - Make sure to first allow 2200/tcp before changing SSH port to 2200. This can be done on the instance Networking/Firewall on the webpage
8. Configure UFW
    - `sudo ufw default deny incoming`
    - `sudo ufw default allow outgoing`
    - `sudo ufw allow 2200/tcp`
    - `sudo ufw allow 123/tcp`
    - `sudo ufw allow 80/tcp`
    - `sudo ufw enable`
### Give grader access
1. create new user grader
    - `sudo adduser grader`
    - you can set a password `passwd grader` and enter password
2. Give `grader` the permission to `sudo`
    - `vim /etc/sudoers`
    - `vim /etc/sudoers.d/grader`,
    - edit the file by typing `grader ALL=(ALL:ALL) ALL` - save and quit.
3. Open terminal on local machine and generate key pair
    - type `ssh-keygen` and save in directory you want.
    - open the directory and copy the content of the file ending in `.pub`. That is the public key
4. On virtual machine type:
    - `su grader` to change user to grader and enter password if you set password earlier
    - `mkdir .ssh` to make directory
    - `vim .ssh/authorized_keys` create a file called authorized_keys and paste the content copied earlier from the .pub file. Save file
    - `chmod 700.ssh`
    - `chmod 644 .ssh/authorized_keys` - to set the right permissions
    - `service ssh restart` to reload SSH

**However, since the above steps were taken on my local machine, `grader` can access the server remotely using the following steps:**
1. Download Private Key below (Private Key not included)
2. Move the PK file into the `.ssh` directory in your home environment like so: `mv ~/<directory-file-downloaded-to/udacity_grader ~/.ssh`
3. change permission - in terminal type: `chmod 600 ~/.ssh/udacity_grader`
4. SSH into the server by typing `ssh grader@35.170.214.60 -p 2200 -i ~/.ssh/udacity_grader` in your terminal
5. Enter SSH password (password not included)
### Prepare to Deploy Project
#### Configure local timezone to UTC
`sudo dpkg-reconfigure tzdata` - Time already set to UTC
#### Install and configure Apache to serve a Python mod_wsgi application
```
sudo apt-get install apache
sudo apt-get install python-setuptools libapache2-mod-wsgi
sudo service apache2 restart
```
#### Install and configure PostgreSQL
1. `sudo apt-get install postgresql` - install postgresql
2. `sudo vim /etc/postgresql/9.5/main/pg_hba.conf` and check that no remote connection is allowed
3. Login as user "postgres" `sudo su - postgres`
4. Enter shell `psql`
5. `CREATE DATABASE catalog` - create database named "catalog"
6. `CREATE USER catalog`; - create user named "catalog"
7. `ALTER ROLE catalog WITH PASSWORD 'password'`;  - set password for user "catalog"
8. `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog`; - grant user "catalog" permission to catalog database
9. `\q` - quit postgreSQL
10. `exit` - exit from user postgres
#### Install git
 - `sudo apt-get install git`
### Deploy the Item Catalog Project
#### Clone and Setup Item Catalog Project
 - type `cd /var/www` to change to the /var/www directory
 - `sudo mkdir CatalogApp` to make a new directory
 - cd into that directory by typing `cd CatalogApp` and type `mkdir CatalogApp` to create another CatalogApp directory
 - cd into that directory and clone the Catalog App using `git clone https://github.com/olamiwhat/catalog-app.git`
 - rename `application.py` to `__init__.py` using sudo mv `application.py __init__.py`
 - open and edit `database_setup.py`, `__init__.py` and `lotsofitem.py` changing engine = create_engine('sqlite:///catalogitemwithusers.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')
 - install pip typing `sudo apt-get install python-pip`
- install psycopg2 by typing `sudo apt-get -qqy install postgresql python-psycopg2`
 - create database schema typing `sudo python database_setup.py`
 - populate database by typing `sudo python lotsofitem.py`
#### Configure and Enable a New Virtual Host
- create a new CatalogApp.conf file: `sudo nano /etc/apache2/sites-available/CatalogApp.conf`
- add the following lines of code to configure the virtual host:
    ```
    <VirtualHost *:80>
                    ServerName 35.170.214.60.xip.io
                    ServerAlias ec2-35-170-214-60.compute-1.amazonaws.com
                    ServerAdmin ubuntu@35.170.214.60
                    WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
                    <Directory /var/www/CatalogApp/CatalogApp/>
                            Order allow,deny
                            Allow from all
                    </Directory>
                    Alias /static /var/www/CatalogApp/CatalogApp/static
                    <Directory /var/www/CatalogApp/CatalogApp/static/>
                            Order allow,deny
                            Allow from all
                    </Directory>
                    ErrorLog ${APACHE_LOG_DIR}/error.log
                    LogLevel warn
                    CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
 - save file and enable the virtual host by typing: `sudo a2ensite CatalogApp`
 #### Create the .wsgi file
 - cd to /var/www/CatalogApp directory `cd /var/www/CatalogApp`
 - create and open the file using `sudo vim catalogapp.wsgi`
 - add the following:
     ```
     #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/CatalogApp/")
    
    from CatalogApp import app as application
    application.secret_key = 'Add_your_super_secret_key'
    ```
#### Restart Apache
    `sudo service apache2 restart`
## Visit the app at the following URL:
**applictaion url: `http://35.170.214.60.xip.io`**
Note the `.xip.io` added to the ip address. That is used to resolve the DNS.
## References
 - https://devops.ionos.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/
 - https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-against-automated-attacks
 - https://flask.palletsprojects.com/en/1.1.x/deploying/mod_wsgi/
 - https://lightsail.aws.amazon.com/ls/docs/en_us/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh
 - https://lightsail.aws.amazon.com/ls/docs/en_us/articles/getting-started-with-amazon-lightsail
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
- http://xip.io/
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

