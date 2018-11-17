# LinuxServerConfigurationAndDeployment
Linux Server Configuration Project 

Project Description
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 13.233.130.123

Accessible SSH port: 2200

Application URL: http://ec2-13-233-130-123.ap-south-1.compute.amazonaws.com/


Create new user named grader and give it the permission to sudo

Steps
1. SSH into the server through ssh -i ~/.ssh/key.peb root@13.233.130.123
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
sudo apt-get install apache2
Install mod_wsgi
Run sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi with sudo a2enmod wsgi
Start the web server with sudo service apache2 start
Clone the Catalog app from Github
Install git using: sudo apt-get install git
cd /var/www
sudo mkdir catalog
Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
cd /catalog
Clone your project from github
Create a catalog.wsgi file, then add this inside:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
Rename application.py to init.py mv application.py __init__.py
Install virtual environment
Install the virtual environment sudo pip install virtualenv
Create a new virtual environment with sudo virtualenv venv
Activate the virutal environment source venv/bin/activate
Change permissions sudo chmod -R 777 venv
Install Flask and other dependencies
Install pip with sudo apt-get install python-pip
Install Flask pip install Flask
Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
Update path of client_secrets.json file
nano __init__.py
Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json
Configure and enable a new virtual host
Run this: sudo nano /etc/apache2/sites-available/catalog.conf
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
Enable the virtual host sudo a2ensite catalog
Install and configure PostgreSQL
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')
python /var/www/catalog/catalog/db_setup.py

Restart Apache
sudo service apache2 restart
Visit site at http://ec2-13-233-130-123.ap-south-1.compute.amazonaws.com/