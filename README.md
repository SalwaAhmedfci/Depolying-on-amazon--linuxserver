
**IP:54.93.240.16**

**SSH Port: 2200**

**Host Name : ec2-54-93-240-16.eu-central-1.compute.amazonaws.com**

**any pass is :salwa**


1-Now move your downloaded udacity.pem key into that folder.

2-make our key secure type $ chmod 600 ~/.ssh/udacity.pem into the terminal.

3-From the terminal type $ ssh -i ~/.ssh/udacity.pem ubuntu@54.93.240.16

4-switch to the root user by typing sudo su -

5-From the command line type $ sudo adduser grader.

6-We must create a file to give the user grader superuser privileges. To do this type $ sudo nano /etc/sudoers.d/grader.
 create a new file that will be the superuser configuration for grader. When nano opens type grader ALL=(ALL:ALL)ALL anad save it .

7-upgrading the current packages, and install new updates with these three commands:
     $ sudo apt-get update
     $ sudo apt-get upgrade
     $ sudo apt-get dist-upgrade
     
8-install a useful tool called Finger with the command $ sudo apt-get install finger.
 it allow us to see the users on this server.

9-From a new terminal run the command: $ ssh-keygen -f ~/.ssh/udacity.rsa

10-In the same terminal we need to read and copy the public key using the command: $ cat ~/.ssh/udacity.rsa.pub.
 Copy the key from the terminal.

11-Run the command $ cd /home/grader to move to the folder.

12-Create a directory called .ssh with the command $ mkdir .ssh

13-Create a file to store the public key with the command $ touch .ssh/authorized_keys

14-Edit that file using $ nano .ssh/authorized_keys and paste in the public key

15-We must change the permissions of the file and its folder by running

$ sudo chmod 700 /home/grader/.ssh

$ sudo chmod 644 /home/grader/.ssh/authorized_keys 

16-Change the owner of the .ssh directory from root to grader by using the command $ sudo chown -R grader:grader /home/grader/.ssh

17-The last thing we need to do for the SSH configuration is restart its service with $ sudo service ssh restart

18-Type $ ~. to disconnect from Amazon Lightsail server

19-login with the grader account using ssh. From your local terminal type $ ssh -i ~/.ssh/udacity.rsa grader@54.93.240.16

20-Lets enforce key authentication from the ssh configuration file by editing $ sudo nano /etc/ssh/sshd_config. 

Find the line that says PasswordAuthentication and change it to no.

Also find the line that says Port 22 and change it to Port 2200. Lastly change PermitRootLogin to no.

21-Restart ssh again: $ sudo service ssh restart

22-  log back through port 2200: $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@54.93.240.16

passphrsa::salwa

23-From here we need to configure the firewall using these commands:

$ sudo ufw allow 2200/tcp

$ sudo ufw allow 80/tcp

$ sudo ufw allow 123/udp

$ sudo ufw enable


**##installing the required software**

$ sudo apt-get install apache2

$ sudo apt-get install libapache2-mod-wsgi python-dev

$ sudo apt-get install git



1-Enable mod_wsgi with the command $ sudo a2enmod wsgi and restart Apache using $ sudo service apache2 restart.

2-We now have to create a directory for our catalog application and make the user grader the owner.

$ cd /var/www

$ sudo mkdir catalog

$ sudo chown -R grader:grader catalog

$ cd catalog



3-In this directory we will have our catalog.wsgi file var/www/catalog/catalog.wsgi,

our virtual environment directory which we will create soon and call venv /var/www/catalog/venv,

and also our application which will sit inside of another directory called catalog /var/www/catalog/catalog.

4-First lets start by cloning our Catalog Application repository by $ git clone [github] catalog

5-Create the .wsgi file by $ sudo nano catalog.wsgi and make sure your secret key matches with your project secret key

import sys

import logging

logging.basicConfig(stream=sys.stderr)

sys.path.insert(0, "/var/www/catalog/")


from catalog import app as application

application.secret_key = 'super_secret_key'



6-Now lets create our virtual environment, make sure you are in /var/www/catalog.

$ sudo pip install virtualenv

$ sudo virtualenv venv

$ source venv/bin/activate

$ sudo chmod -R 777 venv

7-we need to install all packages required for our Flask application. Here are some defaults but you may have more to install.

$ sudo apt-get install python-pip

$ sudo pip install flask

$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 


7-Anywhere in the file where Python tries to open client_secrets.json must be changed to its complete path ex: /var/www/catalog

/catalog/client_secrets.json

8-enable our virtual host to run the site

$ sudo nano /etc/apache2/sites-available/catalog.conf

Paste in the following:

**<VirtualHost *:80>
    ServerName 54.93.240.16
    ServerAlias ec2-54-93-240-16.eu-central-1.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
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
</VirtualHost>**      

9-Enable to virtual host: $ sudo a2ensite catalog.conf and DISABLE the default host $ a2dissite 000-default.conf otherwise your site 

will not load with the hostname.

10-The final step is setting up the database

$ sudo apt-get install libpq-dev python-dev

$ sudo apt-get install postgresql postgresql-contrib

$ sudo su - postgres -i

$ psql

11-Now we create a user to create and set up the database. I name my database catalog with user catalog

$ CREATE USER catalog WITH PASSWORD 'password';

$ ALTER USER catalog CREATEDB;

$ CREATE DATABASE catalog WITH OWNER catalog;

Connect to database $ \c catalog

$ REVOKE ALL ON SCHEMA public FROM public;

$ GRANT ALL ON SCHEMA public TO catalog;

Quit the postgrel command line: $ \q and then $ exit


12-Now use nano again to edit your __init__.py, database_setup.py, and lotsofmenus.py files to change the database engine

from sqlite:// to postgresql://username:password@localhost/catalog

13-Restart your apache server $ sudo service apache2 restart and now your IP address and hostname should both load your application. 
