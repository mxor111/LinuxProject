# Michele Novack Abugosh 

# Project : Linux Server Configuration
This project call to setup and configure a Linux (Ubuntu) web server using Amazon AWS.  The server must be secure and serve and application previously developed in the course (Catalog2.)
The virtual private server is Amazon Lighsail.
The Linux distribution is Ubuntu 18.04 LTS.

# Name	
# IP Address	54.92.179.141
# SSH Port	2200
# Username	grader

# Start a New Ubuntu Linux server Instance
•	Login into Amazon Lightsail using an Amazon Web Services account.
•	Once you are login into the site, click Create instance.
•	Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
•	Choose a instance plan (I took the cheapest, $5/month).
•	Keep the default name provided by AWS or rename your instance.(FunnyApp)
•	Click the Create button to create the instance.
•	Wait for the instance to start up.
https://lightsail.aws.amazon.com/ls/webapp/us-east-1/instances/FunnyApp/connect

# Get Instance connected to Remote Server: 

•	From the Account page for AWS Lightsail instance : click SSH Keys
•	Download the Key
•	Save the key to your home drive LightsailPrivateKey.pem
•	Open your terminal  and cd into your home drive.  Then change permission to the downloaded key 
•	In terminal type chmod 600 LightsailDefaultKey.pem
•	Type:  mv LightsailDefaultKey.pem grader_key.rsa
•	Check to see if connected   54.92.179.141 from my AWS instance
•	Run the following command on locally:   grader -I grader grader@54.92.179.141. SAY YES

# To connect to EC2 instance you need the grader KEY:( supplied separately in the submit process)

# ssh -i grader grader@54.92.179.141

# Secure the Server by checking for updates:  

sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade

# Create New User:  Create a new user named grader and grant this user sudo access permissions. You must be logged into ubuntu to add user. 

sudo adduser grader

password: grader

# GRANT (sudo) permission to the grader by editing the /etc/sudoers.d/  directory 

sudo touch /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader

grader ALL=(ALL:ALL)ALL  - close and save

# Update All Currently installed applications:

sudo apt-get update
sudo apt-get upgrade

# Configure the local timezone to UTC:  select OK for more option and select UTC save
# Resource: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442

sudo dpkg-reconfigure tzdata

# Set time to sync with NTP and add additional servers to the /etc/ntp.config file

sudo apt-get install ntp
sudo service ntp reload (restart the service)

# Install Python Applications:

Sudo apt-get install python-psycopg2
Sudo apt-get install python-flask python-sqlalchemy
Sudo apt-get install python-pip

# Verify grader has sudo permissions: in ubuntu run

su – grader

# Resource: DigitalOcean, How To Add and Delete Users on an Ubuntu 14.04 VPS

# Create .ssh directory  Added file .ssh/authorized_keys and copy the public key contents of the grader.pub to authorized_keys and finally restrict permission to the .shh and authorized_key

cd /home /grader
mkdir .ssh
sudo touch .ssh/authorized_keys
sudo nano .ssh/authorized_keys

# Open Local Terminal and Generate a SSH key pair : Public Key

ssh-keygen
Name it – grader
Passphrase - grader
less grader.pub (will give you the key -  See Appendix for public Key)
 
# Add key pair to .ssh/authorized_keys  file created 

sudo chown grader:grader .ssh
chmod 700 .ssh
chmod 664 .ssh/authorized_keys

# Restart SSH  : sudo service ssh restart

# Test your sign in: PWD - GRADER       ssh -i grader grader@54.92.179.141 

# Resource: DigitalOcean, How To Set Up SSH Keys.         Ubuntu Wiki, SSH/OpenSSH/Keys

# Automatically install updates: The unattended-upgrades package can be used to automatically install important system updates. Enable upgrades:

sudo apt-get install unattended-upgrades

sudo dpkg-reconfigure --priority-low unattended-upgrades

sudo service apache2 restart

# Resource: Official Ubuntu Documentation, Automatic Updates.

# Only allow Key Based Authentication   

•	Edit the /etc/ssh/sshd_config file  

sudo nano /etc/ssh/sshd_config
•	Change  the following line:  # What port, IPs and protocols we listen for :  Port 22  to Port 2200
•	Remove Root Login:  change the following line # Authentication : PermitRootLogin  yes to NO
•	Force SSH Login: change the following line # Change to no to disable tunneled clear text passwords :  PasswaordAuthentication yes to NO

# Restart SSH :  sudo service ssh restart
NOTE: Be sure to have the ability to log in to grader before logging out of root.

# Configure the Firewall (UFW)  to only allow incoming connect for SSH(port 2200), HTTP (port 80) and NTP (port 123)
# To set up the UFW – check the firewall status

sudo ufw status

sudo ufw deny incoming
sudo ufw allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp

# Then ENABLE UFW
sudo ufw enable
 # Then Exit the SSH Connection :   exit

# Update  Amazon Lightsail Instance:
•	Login into AWS Lightsail instance, click on Networking tab , and then change the Firewall configuration to match.
 
Official Ubuntu Documentation, UFW - Uncomplicated Firewall.
Now you are able to log in using :   ssh -grader grader@54.92.179.141
# Reference Documents : DigitalOcean, Reddit.

# Install Apache2 HTTP Server:
sudo apt update
sudo apt-get install apache2

# In your browser, type http://54.92.179.141 The Apache Default page will appear. 

# Install Python mod_wsgi
Sudo apt-get install libapache2-mod-wsgi
Sudo apt-get install python-setuptools libapache2-mod-wsgi
Sudo service apache2 start

# Check to see if working + +Open any browser:  paste your IP  to see if working you should see  default apache 2 page.  

# Install PostgreSQL and Configure – While logged in as grader

sudo apt-get install libpq-dev python dev
sudo apt-get install postgresql postgresql-contrib
Switch to postgres user and open terminal;
sudo -u postgres psql

# Create a catalog2 User with a password and give them ability to create databases: 

CREATE USER catalog2 with PASSWORD ‘grader’;
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog2 WITH OWNER catalog2;
CREATE DATABASE itemcatalogdb;
\c catalog2
GRANT ALL PRIVILEGES on DATABASE catalog2 to catalog2;
\q
exit

# Check No remote connection to the database are allowed: 
Sudo nano /etc/postgresql/9.3/main/pg_hba.conf

# Restart Apache2:     sudo service apache2 restart

# Install GIT – log in as grader  : Install Git  so that you can clone the Catalog2 app from github

sudo apt-get install git

# Clone Udacity Project 2 catalog2 app to the AWS server
# Create a folder inside  the /var/www folder called “catalog2” and then CD into this folder(this is python flask app and not just html)

cd /var/www
Make the directory and name it catalog2
sudo mkdir catalog2
sudo chown -R grader:grader catalog2 (change ownership from catalog2 to grader)

cd catalog2

# Clone your project from github 
git clone https://github.com/mxor111/catalog2.git catalog2

 # The project is now at /var/www/catalog2/catalog2
# Reference Documentation: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible

# Edit Client Secret. Connect to your project in Google API credentials and add the IP address as Authorized redirect URIs and Authorized JavaScript origins.
# Note: When you set up OAuth for your application, you will need a DNS name that refers to your instance's IP address. You can use the xip.io service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name 54.92.179.141.xip.io refers to the server above

# Update the client secret
cd /var/www/catalog2
sudo touch client_secrets.json
sudo nano client_secrets.json

# Update Google
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog2/client_secrets.json', scope='')

# Google API will not accept Public IPs .  You need to have a domain name -  Once you got your domain name add 54.92.179.141.xip.io .tp Virtual Host
# Authenticate Login through Google:
•	Go to Google Cloud .
•	Click APIs & services on left menu.
•	Click Credentials.
•	Create an OAuth Client ID (under the Credentials tab), and add http://55.92.179.141 and http://ec2-54-92.179.141.us-east-1.compute.amazonaws.com as authorized JavaScript origins.
•	Add http://ec2-54.92.179.141.us-east-1.compute.amazonaws.com/oauth2callback as authorized redirect URI.
•	Download the corresponding JSON file, open it et copy the contents.
•	Open /var/www/catalog2/catalog2/client_secret.json and paste the previous contents into the this file.
•	Replace the client ID of the templates/login.html file in the project directory.

# Install Flask and dependencies for deploying on Ubuntu:

sudo pip install flask

# Install App Dependencies for Flask and Database
sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo pip install flask-seasurf
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests

# Install VIRTUAL MACHINE: Set up a virtual environment to keep the application and it dependencies isolated from the main system. 
# Run this if you have not already install in your python environment:

sudo apt-get install python-pip

# Create a Virtual Environment for Flask Applications . Using pip to install flask
sudo apt-get install python-virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod 777 venv

# Configure and Enable Vitrual Host:

sudo nano /etc/apache2/sites-available/catalog2.conf

# Add the following lines of code to the file to configure the virtual hose.  Be sure to change the ServerName to your domain or cloud server’s IP address:

<VirtualHost *:80>
     ServerName 54.92.179.141
     ServerAlias www.54.92.179.141.xip.io
ServerAdmin mishtay@yahoo.com
      WSGIDaemonProcess application user=grader thread=3
WSGIScriptAlias / /var/www/catalog2/catalog2.wsgi
<Directory /var/www/catalog2/>
       WSGIProcessGroup application
       WSGIApplicationGroup %{GLOBAL}
       Require all granted
</Directory>
     Alias /static /var/www/catalog2/static
<Directory /var/www/catalog2/static/>
Order allow,deny
Allow from all
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# ENABLE the Virtual Host:

sudo a2ensite catalog2
# Restart Apache2 Server :     sudo service apache2 reload

# Create a catalog.wsgi file, then add this inside

sudo touch /var/www/catalog2/catalog2.wsgi

sudo nano catalog.wsgi

# Add the following content:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog2/catalog2/")
sys.path.insert(1, /var/www/catalog2/”)

from catalog import app as application
application.secret_key = 'supersecretkey'

# Restart Apache :  sudo service apache2 restart

# Update Application to accommodate the change:  app.py and database_setup.py 

sudo nano app.py

CLIENT_ID = json.loads(open(‘/var/www/catalog2/client_secrets.json’. ‘r’.read())[‘web’][‘client_id’]

engine = create_engine('postgresql:///var/www/catalog/catalog2/itemscatalog.db',)


CHECK WITH MICHAEL##(‘postgresql://catalog2:grader@localhost/catalog’)

sudo nano database_setup.py
engine = create_engine('postgresql:///var/www/catalog/catalog2/itemscatalog.db',

### ASK MICHAEL  create_engine(‘postresql://catalog2:grader@localhost/catalog2’)

# Reference Documentation:  https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
# https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
# https://help.ubuntu.com/community/PostgreSQL

# Update Facebook
app_id = json.loads(open('/var/www/catalog/catalog2/fb_client_secrets.json', 'r').read())[
'web']['app_id']

app_secret = json.loads(open('/var/www/catalog/catalog2/fb_client_secrets.json', 'r').read())[
'web']['app_secret']

# Additional Security
Add feature to protect against attackers, I installed fail2ban a software package that blocks up address with multiple failed login attempts within a certain about of time 
# To install fail2ban :   

sudo apt-get update
sudo apt-get install fail2ban

# Configuring fail2ban was simple, the default configurations settings were adequate.  I had to copy the default configuration file to new file to prevent updates from overwriting my configurations setting:

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
under [sshd] change port
port = 2200
sudo service fail2ban restart

# Reference: 	DigitalOcean, How To Protect SSH with Fail2Ban on Ubuntu 14.04.
# Fail2Ban Official website.

# Debug
If you are getting errors, you can check out Apache's error log for debugging:
$ sudo tail /var/log/apache2/error.log

# Resources:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html
https://www.digitalocean.com/docs/droplets/resources/troubleshooting-ssh/connectivity/
