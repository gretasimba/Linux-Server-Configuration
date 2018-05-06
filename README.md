# Linux-Server-Configuration
This is Linux server installation and preparation to host my Catalog application. I secured the server from a number of attack vectors, installed and configured a Postgres, and deploeyed application onto it.The EC2 URL is 
http://ec2-18-220-63-1.us-east-2.compute.amazonaws.com and the local IP address is http://18.220.63.1/

# Ubuntu Linux server
Started a new Ubuntu Linux server instance on Amazon Lightsail.
Follow instructions on Udacity Project "Get started on Lightsail" page
Created "ubuntu-olga-1" instance.
Download SSH .pem file and saved it in Vagrant folder

# Secure Server
Update and Install new versions of all currently installed packages:
 
	$ sudo apt-get update
	
	$ sudo sudo apt-get upgrade
 
Change the SSH port from 22 to 2200
Set Uncomplicated Firewall (UFW) to only allow incoming connec53
 edit /etc/ssh/sshd_config
 Changed port to 2200 
 Changed PermitRootLogin from without-password to no
 configure the Lightsail: added Custom TCP 2200 in Networking tab

# Give grader accesstions for SSH (port 2200), HTTP (port 80), and NTP (port 123):

	$  sudo ufw status (verify inactive)
  	$  sudo ufw default deny incoming
	$  sudo ufw default allow outgoing
 	$  sudo ufw allow 2200/tcp
 	$  sudo ufw allow www
 	$  sudo ufw allow ntp
 	$  sudo ufw enable
  
# SSH to Server from local Vagrant virtual machine
SSH into the "olga-first-linux" instance, specify the path to .pem (SSH) file, the port 2200, the instance user ubuntu and ip address:
  
	$ ssh -i PK.pem -p 2200 ubuntu@18.220.63.1
 
Create a new user account named grader with password grader.
  
	$ sudo adduser grader

Give grader the permission to sudo.
 Copy 
	 
	$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
 And Replace ubuntu with grader using nano editor:
  
	$ sudo nano /etc/sudoers.d/grader
 type in grader ALL=(ALL:ALL) ALL, save and quit
 
 # Set ssh login using keys
 generate keys on local machine:
  
	$ ssh-keygen
 save the private key in ~/.ssh on local machine
 deploy public key on developement enviroment

 On LightSail machine:
  
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
 
 copy the public key generated on your local machine to this file and save
  
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
 
 reload SSH using service ssh restart
 now  use ssh to login with the new user you created
 from local machine run:
 	
	ssh -i [privateKeyFilename] -p 2200 grader@18.220.63.1
 
 # Update and upgrade all currently installed packages
  Update the list of available packages and their versions:
		
	$ sudo apt-get update
  Install newer vesions of packages you have:
    
	$ sudo sudo apt-get upgrade
 
 # Configure the local timezone to UTC
 Run

	$ sudo dpkg-reconfigure tzdata
 Select none of the above, then UTC, click OK
 
 # Install and configure Apache to serve a Python mod_wsgi application
 Install Apache web server:
  
	$ sudo apt-get install apache2
 Open browser window and run 18.220.63.1 - it should show Ubuntu home page
 Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
  
	$ sudo apt-get install python-setuptools libapache2-mod-wsgi
 Restart the Apache server for mod_wsgi to load:
  
	$ sudo service apache2 restart
 Extend Python with additional packages that enable Apache to serve Flask applications:
  
	$ sudo apt-get install libapache2-mod-wsgi python-dev
 Enable mod_wsgi:
	
	$ sudo a2enmod wsgi
 Move to the www directory:
  
	$ cd /var/www
 Setup a catalog directory for the app:
	
	$ sudo mkdir catalog
	$ cd catalog
	$ sudo mkdir catalog
 Create the file that will contain the flask application logic:
	
	$ sudo nano __init__.py
 Install pip installer:
  
	$ sudo apt-get install python-pip
 Install virtualenv:
  
	$ sudo pip install virtualenv
 Set virtual environment to name 'venv':
  
	$ sudo virtualenv venv
 Enable all permissions for the new virtual environment:
  
	$ sudo chmod -R 777 venv
 Activate the virtual environment:
  
	$ source venv/bin/activate
 Install Flask inside the virtual environment:
  
	$ pip install Flask
 Create a virtual host config file
  
	$ sudo nano /etc/apache2/sites-available/catalog.conf
 Paste the following:
 
 	<VirtualHost *:80>
    	  ServerName 18.220.63.1
    	  ServerAdmin admin@18.220.63.1
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
 
 Enable the virtual host:
    
	$ sudo a2ensite catalog
 Create wsgi file:
    
	$ cd /var/www/catalog
	$ sudo vim catalog.wsgi
 Paste in the following lines of code:
 
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'super_secret_key'
  
 Restart Apache:
   
	$ sudo service apache2 restart
Install Git:
   
	$ sudo apt-get install git

# Clone GitHub repository and make it web inaccessible
  Clone project Catalog-Item-Project solution repository on GitHub:
    
	$ git clone https://github.com/gretasimba/Catalog-Item-Project.git
  Move all content of Catalog-Item-Project created  directory to /var/www/catalog/catalog/.
  Make the GitHub repository inaccessible:
  Source: Stackoverflow
  Create and open .htaccess file:
    
	$ cd /var/www/catalog/ and $ sudo vim .htaccess
  Paste in the following:
    RedirectMatch 404 /\.git
  copy content of application.py to __init__.py

# Install needed modules & packages
  Activate virtual environment:
    
	$ source venv/bin/activate
  Install httplib2 module in venv:
    
	$ pip install httplib2
  Install requests module in venv:
		
	$ pip install requests
  Install oauth2client.client:
    
	$ sudo pip install --upgrade oauth2client
  Install SQLAlchemy:
    
	$ sudo pip install sqlalchemy
  Install the Python PostgreSQL adapter psycopg:
   
	$ sudo apt-get install python-psycopg2

# Install and configure PostgreSQL
   Install PostgreSQL:
     
	$ sudo apt-get install postgresql postgresql-contrib
   Open the database setup file:
    
	$ sudo vim database_setup.py
   Replace the line "engine = create_engine('sqlite:///adaptCntr.db')"
    with "engine = create_engine('postgresql://catalog:catalog@localhost/catalog')"
   Change the same line in  __init__.py respectively
   Create needed linux user for psql:
     $ sudo adduser catalog add password catalog
   Change to default user postgres:
     $ sudo su - postgres
   Connect to the system:
     $ psql
   "#" stands for the command prompt in psql 
   Create user with LOGIN role and set a password:
    # CREATE USER catalog WITH PASSWORD 'catalog';
   Allow the user to create database tables:
    # ALTER USER catalog CREATEDB;
   Create database:
    # CREATE DATABASE catalog WITH OWNER catalog;
   Connect to the database catalog # \c catalog
   Revoke all rights:
    # REVOKE ALL ON SCHEMA public FROM public;
   Grant only access to the catalog role:
    # GRANT ALL ON SCHEMA public TO catalog;
   Exit out of PostgreSQl and the postgres user:
    # \q, then $ exit
   Create postgreSQL database schema:
     $ python database_setup.py
   Fill database with data:
   $ sudo vim database_insert_data.py
   Replace the line "engine = create_engine('sqlite:///adaptCntr.db')"
    with "engine = create_engine('postgresql://catalog:catalog@localhost/catalog')"
   $ python database_insert_data.py
   
   # Run application
   Restart Apache:
     
     $ sudo service apache2 restart
   Open a browser and put in public ip-address as url
   18.220.63.1
   the application should come up
   check the errors in error log file:
     
     $ sudo tail -20 /var/log/apache2/error.log
     
  # Get OAuth-Logins Working
   Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address:
   for it is ec2-18-220-63-1.us-east-2.compute.amazonaws.com
   Open the Apache configuration files for the web app: $ sudo vim /etc/apache2/sites-available/catalog.conf
    Paste in the following line below ServerAdmin:
    ServerAlias http://ec2-18-220-63-1.us-east-2.compute.amazonaws.com
    Enable the virtual host:
    
    $ sudo a2ensite catalog
   To get the Google+ authorization working:
   Go to the project on the Developer Console: https://console.developers.google.com/project
   Navigate to APIs & auth > Credentials > Edit Settings
   add http://ec2-18-216-114-52.us-east-2.compute.amazonaws.com and http://18.220.63.1 to Authorized JavaScript origins 
   and http://ec2-18-216-114-52.us-east-2.compute.amazonaws.com/gconnect to Authorized redirect URIs 
   
   # Resources
   https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04
   https://help.ubuntu.com/community/PostgreSQL
   https://github.com/stueken/FSND-P5_Linux-Server-Configuration
   https://discussions.udacity.com/t/oauth-provider-callback-uris/20460
   https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
   https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
   https://help.github.com/articles/set-up-git/#platform-linux
