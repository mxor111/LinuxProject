Michele Novack Abugosh 

Project : Linux Server Configuration
This project call to setup and configure a Linux (Ubuntu) web server using Amazon AWS.  The server must be secure and serve and application previously developed in the course (Catalog) . Prepare it to host your web application to include installing updates, securing it from attacks. 
Name	Value
IP Address	35-153.129.220
SSH Port	2200
Username	grader
URL of Applicaiton	https://lightsail.aws.amazon.com/ls/remote/us-east-1/instances/CatalogApp/terminal?protocol=ssh


To connect to EC2 instance you need the grader ( supplied separately in the submit process)

ssh -i grader grader@35.153.129.220

Launch your EC2 Terminal , Then access the EC2 instance using the following command: 

ssh -i grader  grader@35.153.129.220

Check for updates:  

sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade


Create New User:  Create a new user named grader and grant this user sudo access permissions

sudo adduser grader

Followed the instructions in command line and added a secure password. After that I GRANTED (sudo) permission to the grader by editing the /etc/sudoers.d/  directory 

Sudo nano /etc/sudoers.d/grader

grader ALL-(ALL:ALL)ALL  - close and save


Update All Currently installed applications:

Sudo apt-get update
Sudo apt-get upgrade

Configure the local timezone to UTC:  select OK for more option and select UTC save
 http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442

Sudo dpkg-reconfigure tzdata

Set time to sync with NTP and add additional servers to the /etc/ntp.config file


Sudo apt-get install ntp

Restart NTP service

Sudo service ntp reload

ADD Python environment


Sudo apt-get install python-psycopg2
Sudo apt-get install python-flask python-sqlalchemy
Sudo apt-get install python-pip


Change to Grader:  


su – grader

Create .ssh directory  
Added file .ssh/authorized_keys and copy the public key contents of the grader.pub to authorized_keys and finally restrict permission to the .shh and authorized_key

mkdir .ssh
touch /ssh/authorized_keys
sudo nano .ssh/authorized_keys

Open Local Terminal and Generate a Authorized Key:

ssh-keygen
Name it - grader
less grader.pub – will give you the key


Chmod 700 .ssh
hmod 644 .ssh/authorized_keys

Restart SSH 

sudo service ssh restart


Test your sign in:

ssh -i grader  grader@35.153.129.220

Only allow Key Based Authentication
To force key based authentication edit the /etc/ssh/sshd_config file 

sudo nano /etc/ssh/sshd_config

Update the following field # Change to no to disable tunnelled clear text passwords

PasswordAuthentication yes 
TO :
PasswordAuthentication  NO	

Then, restart the ssh service
sudo service ssh restart 

The SSH is hosted on non-default port -  Change the Default to host on Port 2200 - edit the /etc/ssh/sshd_config file
Port 22 
TO
Port 2200
The Restart the Service: 
sudo service ssh restart
# in AWS Lightsail update the Security Group , add port 2200 as the inbound custom TCP rule port
Configure the Firewall (UFW)  to only allow incoming connect for SSH(port 2200), HTTP (port 80) and NTP (port 123)
To set up the UFE , first check the firewall status:  it should show inactive
sudo ufw status

Establish the rules for SSH (port 2200) – 
Sudo ufw allow 2200/tcp

For HTTP (port 80)
sudo ufw allow www

And for NTP (port 123)
sudo ufw allow ntp

Then ENABLE UFW
sudo ufw enable

Check to see if Active:
sudo ufw status

DISABLE remote login of the root user.  Edit the /etc/ssh/sshd_config file and update the following:
sudo nano /etc/ssh/sshd_config
Edit the following
PermitRootLogin with-out-password
TO
PermitRootLogin NO

Then Restart the SHH server:
Sudo service ssh restart

Now you are able to log in using :   ssh -grader grader@3.84.147.247
Additional Security
Add feature to protect against attackers, I installed fail2ban a software package that blocks up address with multiple failed login attempts within a certain about of time 
To install fail2ban :   
sudo apt-get update
sudo apt-get install fail2ban

Configuring fail2ban was simple, the default configurations settings were adequate.  I had to copy the default configuration file to new file to prevent updates from overwriting my configurations setting:
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

S
sudo service fail2ban restart

Reference Documents : DigitalOcean, Reddit.

INSTALL PACKAGES FOR SERVER NEEDS
Apache2 HTTP Server:
sudo apt-get install apache2

mod_wsgi
Sudo apt-get install libapache2-mod-wsgi
Sudo apt-get install python-setuptools libapache2-mod-wsgi
Sudo service apache2 start

Check to see if working + +Open any browser:  paste your IP  to see if working you should see  default apache 2 page. 
Install GIT - good
Install Git  so that you can clone the Catalog2 app from github
Sudo apt-get install git

Clone Udacity Project 2 catalog2 app to the AWS server
Create a folder inside  the /var/www folder called “catalog2” and then CD into this folder(this is python flask app and not just html)
cd /var/www
Make the directory and name it catalog2
sudo mkdir catalog
sudo chown -R grader:grader catalog
sudo chmod catalog
cd catalog
Change the owner of the created direcrtory
Clone your project from github 
git clone https://github.com/mxor111/catalog2.git catalog2

 The project is now at /var/www/catalog2/catalog2

Reference Documentation: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
Create a catalog.wsgi file, then add this inside
sudo nano catalog.wsgi
Create the following:   and save
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog2/")

from catalog import app as application
application.secret_key = 'supersecretkey'

Install VIRTUAL MACHINE: 
Flask - 
Set up a virtual environment to keep the application and it dependencies isolated from the main system. Create a Virtual Environment for Flask Applications . Using pip to install flask
Run this if you have not already install in your python environment:
sudo apt-get install python-pip
sudo apt-get install python-virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod 777 venv


Install Flask and dependencies:
sudo pip install flask
Install App Dependencies for Flask and Database
sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo pip install flask-seasurf
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests

Update path to client_secrets.json file 
sudo nano app.py

change path : /var/www/catalog/catalog2/client_secrets.json

Configure and Enable Vitrual Host:
sudo nano /etc/apache2/sites-available/catalog.conf

Add the following lines of code to the file to configure the virtual hose.  Be sure to change the ServerName to your domain or cloud server’s IP address:

<VirtualHost *:80>     ServerName Your-Public-IP-Address     ServerAdmin Your-prefrred-email-address     WSGIScriptAlias / /var/www/catalog/catalog.wsgi     <Directory /var/www/catalog/catalog/>         Order allow,deny         Allow from all     </Directory>     Alias /static /var/www/catalog/catalog/static     <Directory /var/www/catalog/catalog/static/>         Order allow,deny         Allow from all     </Directory>     ErrorLog ${APACHE_LOG_DIR}/error.log     LogLevel warn     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
ENABLE the Virtual Host:
sudo a2ensite catalog
Reload the server
Sudo service apache2 reload


Install PostgreSQL and Configure
sudo apt-get install libpq-dev python dev
sudo apt-get install postgresql postgresql-contrib

sudo  su – postgres
psql


Create a User 
CREATE USER catalog2 with PASSWORD ‘password’;
ALTER USER catalog CREATEDB
CREATE DATABASE catalog WITH OWNER catalog2
\c catalog
GRANT ALL PRIVILEGES on DATABASE catalog2 to catalog2;
\q
exit

Reference Documentation:  https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
https://help.ubuntu.com/community/PostgreSQL

Update Application to accommodate the change:  app.py and database_setup.py 
Change your create_engine line in your application.py(app.py) file 
sudo nano app.py

engine = create_engine('sqlite:///var/www/catalog/catalog2/itemscatalog.db',)

Update 
sudo nano database_setup.py

engine = create_engine('sqlite:///var/www/catalog/catalog2/itemscatalog.db',

Update Facebook
app_id = json.loads(open('/var/www/catalog/catalog2/fb_client_secrets.json', 'r').read())[
'web']['app_id']

app_secret = json.loads(open('/var/www/catalog/catalog2/fb_client_secrets.json', 'r').read())[
'web']['app_secret']


Update Google
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog2/client_secrets.json', scope='')

