# Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity.

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

*IP ADDRESS: 3.14.255.28 
* SSH PORT: 2200
## Start a new Ubuntu Linux Server instance on Amazon Lightsail
1. Create an AWS account
2. Click **Create instance** button on the home page
3. Select **Linux/Unix** platform
4. Select **OS Only** and **Ubuntu** as blueprint
5. Select an instance plan
6. Name your instance
7. Click **Create** button

## SSH into your Server
1. Download private key from the **SSH keys** section in the **Account** section on Amazon Lightsail. The file name should be like _LightsailDefaultPrivateKey-us-east-2.pem_
2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine
3. Copy and paste content from downloaded private key file to **lightsail_key.rsa**
4. Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`

## Update all currently installed packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages

##  Change the SSH port from 22 to 2200
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH: `$ sudo service ssh restart`

## Configure the firewall
1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow 80/tcp`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Enable firewall: `$ sudo ufw enable`
8. Check out current firewall status: `$ sudo ufw status`
9. Update the firewall configuration on Amazon Lightsail website under **Networking**. Add **port 80, 123, 2200**
10. Open up a new terminal and you can now ssh in via the new port 2200: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@3.14.255.28 -p 2200`

## Create a new user account **grader** and give **grader** sudo access
1. Create a new user account **grader**:`$ sudo adduser grader`
2. Create a file named grader under this path: `$ sudo touch /etc/sudoers.d/grader`
3. Edit this file: `$ sudo nano /etc/sudoers.d/grader`, add code `grader ALL=(ALL:ALL) ALL`. Save and exit

## Set SSH login using keys
1. Create an SSH key pair for **grader** using the `ssh-keygen` tool on your local machine. Save it in `~/.ssh` path
2. Deploy public key on development environment
    * On your local machine, read the generated public key
     `cat ~/.ssh/FILE-NAME.pub`
    * On your virtual machine
   ```$ su - grader
      $ cd /home
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ nano .ssh/authorized_keys
      ```
    * Copy the public key to this _authorized_keys_ file on the virtual machine and save
3. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` on your virtual machine to change file permission
4. Restart SSH: `$ sudo service ssh restart`
5. Now you are able to login in as grader: `$ ssh -i ~/.ssh/grader_key -p 2200 grader@3.14.255.28`
6. Restart SSH: `$ sudo service ssh restart`

## Configure the local timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Choose **None of the above** to set timezone to UTC
3. UTC is already set

## Install and configure Apache
1. Install **Apache**: `$ sudo apt-get install apache2`
2. Go to http://3.14.255.28/, if Apache is working correctly, a **Apache2 Ubuntu Default Page** will show up

## Install and configure Python mod_wsgi
1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
3. Restart **Apache**: `$ sudo service apache2 restart`
4. Check if Python is installed: `$ python`

## Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
4. Check to make sure it looks like this:
   ```
   # Database administrative login by Unix domain socket
   local   all             postgres                                peer

   # TYPE  DATABASE        USER            ADDRESS                 METHOD

   # "local" is for Unix domain socket connections only
   local   all             all                                     peer
   # IPv4 local connections:
   host    all             all             127.0.0.1/32            md5
   # IPv6 local connections:
   host    all             all             ::1/128                 md5
   ```
## Create new PostgreSQL user called **catalog**
1. Switch to PostgreSQL defualt user **postgres**: `$ sudo su - postgres`
2. Get into postgreSQL shell `psql`
3. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
   
   ```
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   ```
4. Set a password for user catalog
   
   ```
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   ```
5. Give user "catalog" permission to "catalog" application database
   
   ```
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```
6. Quit postgreSQL `postgres=# \q`
7. Exit from user "postgres" 
   
   ```
   exit
   ```
## Install git and clone catalog application from github
1. Run `$ sudo apt-get install git`
2. Create directory: `$ mkdir /var/www/catalog`
3. CD to this directory: `$ cd /var/www/catalog`
4. Change the ownership: `$ sudo chown -R ubuntu:ubuntu catalog/`
5. Clone the catalog app: `$ sudo git clone https://github.com/RennerLima/Item_Catalog.git catalog`
6. CD to `/var/www/catalog/catalog`
7. Change file **project.py** to **__init__.py**: `$ mv project.py __init__.py`
8. Change line `app.run(host='0.0.0.0', port=5000)` to `app.run()` in **__init__.py** file

## Edit client_secrets.json file
1. Modify __init__.py at `CLIENT_ID = json.loads(open('client_secrets.json','r').read())['web']['client_id']` **to** `CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json')`

## Setup for deploying a Flask App on Ubuntu VPS
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```

## Setup and enble a virtual host
1. Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName 3.14.255.28
		ServerAdmin renneraslima@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Run `$ sudo a2ensite catalog` to enable the virtual host
4. Restart **Apache**: `$ sudo service apache2 reload`

## Configure .wsgi file
1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'super_secret_key'
```
3. Restart **Apache**: `$ sudo service apache2 reload`

## Edit the database path
1. Replace lines in `__init__.py`, `database_setup.py`, and `categories.py` with `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

## Disable defualt Apache page
1. `$ sudo a2dissite 000-defualt.conf`
2. Restart **Apache**: `$ sudo service apache2 reload`

## Set up database schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python categories.py`
3. Restart **Apache**: `$ sudo service apache2 reload`
4. Now follow the link to http://3.14.255.28/  the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)

## Sources
1. [Amazon Lightsail Website](https://aws.amazon.com/lightsail/?p=tile)
2. [Google API Concole](https://console.cloud.google.com/)
3. [Udacity](https://www.udacity.com)
4. [Apache](https://httpd.apache.org/docs/2.2/configuring.html)












