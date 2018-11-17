# LinuxServerConfigurationAndDeployment
Linux Server Configuration Project 

Project Description
Create an instance of Basic Unix based OS from lightsail AWS server and deploy your Item catalog Project into the server. It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 13.233.130.123

Accessible SSH port: 2200

Application URL: http://ec2-13-233-130-123.ap-south-1.compute.amazonaws.com/


Create new user named grader and give it the permission to sudo

Steps
1. SSH into the server through ssh -i ~/.ssh/key.peb ubuntu@13.233.130.123
2. Run $ sudo adduser grader to create a new user named grader
3. Create a new file in the sudoers directory with sudo nano /etc/sudoers.d/grader
4. Add the following text grader ALL=(ALL:ALL) ALL

Update all currently installed packages
1. Download package lists with sudo apt-get update
2. Fetch new versions of packages with sudo apt-get upgrade


Change SSH port from 22 to 2200
1. Run sudo nano /etc/ssh/sshd_config
2. Change the port from 22 to 2200
3. Confirm by running ssh -i ~/.ssh/key.peb -p 2200 root@13.233.130.123

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Before doing this step make sure that in Lightsail server firewall you have enable 2200 port. Otherwise you will be locked out of your machine. 
1. sudo ufw allow 2200/tcp
2. sudo ufw allow 80/tcp
3. sudo ufw allow 123/udp
4. sudo ufw enable

Configure the local timezone to UTC
1. Run sudo dpkg-reconfigure tzdata and then choose UTC

Configure key-based authentication for grader user
1. Run this command sudo nano /home/grader/.ssh/authorized_keys
2. Copy your public key into this file.

Disable ssh login for root user
1. Run sudo nano /etc/ssh/sshd_config
2. Change PermitRootLogin without-password line to PermitRootLogin no
Restart ssh with sudo service ssh restart
Now you are only able to login using ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.233.130.123
Note : Check the entries first , You might not need to disable in Lightsail Server , it has already
been disabled.

Install Apache

1. sudo apt-get install apache2

Install mod_wsgi
1. Run sudo apt-get install libapache2-mod-wsgi python-dev
2.  Enable mod_wsgi with sudo a2enmod wsgi
3. Start the web server with sudo service apache2 start

4. cd /var/www
5. sudo mkdir catalog
6. Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
7. cd /catalog
8. Clone your project from github
9. Create a catalog.wsgi file, then add this inside:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
Rename application.py to init.py mv application.py __init__.py


Install virtual environment
1. Install the virtual environment creator sudo pip install virtualenv
2. Create a new virtual environment with sudo virtualenv venv
3. Activate the virutal environment source venv/bin/activate
4. Change permissions sudo chmod -R 777 venv

Install Flask and other dependencies
1. Install pip with sudo apt-get install python-pip
2. Install Flask pip install Flask
3. Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

Update path of client_secrets.json file
1. sudo nano __init__.py
2. Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json


Configure and enable a new virtual host
1. Run this: sudo nano /etc/apache2/sites-available/catalog.conf

Paste this code:
<VirtualHost *:80>
    ServerName 13.233.130.123
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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

2. Enable the virtual host sudo a2ensite catalog


Install and configure PostgreSQL
1. sudo apt-get install libpq-dev python-dev
2. sudo apt-get install postgresql postgresql-contrib
3. sudo su - postgres
4. psql
5. CREATE USER catalog WITH PASSWORD 'password';
6. ALTER USER catalog CREATEDB;
7. CREATE DATABASE catalog WITH OWNER catalog;
8. REVOKE ALL ON SCHEMA public FROM public;
9. GRANT ALL ON SCHEMA public TO catalog;

Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')

10. python /var/www/catalog/catalog/db_setup.py

Restart Apache
1. sudo service apache2 restart
Visit site at http://ec2-13-233-130-123.ap-south-1.compute.amazonaws.com/