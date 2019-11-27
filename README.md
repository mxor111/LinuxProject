# LinuxProject
Michele Novack Abugosh 

# Project : Linux Server Configuration
This project call to setup and configure a Linux (Ubuntu) web server using Amazon AWS.  The server must be secure and serve and application previously developed in the course (Catalog2.)
The virtual private server is Amazon LightSail.
The Linux distribution is Ubuntu 18.04 LTS.

# ssh -i grader grader@54.198.244.135 -p 2200

# Name	
IP Address	54.198.244.135
SSH Port	2200
Username	Grader

# Start a New Ubuntu Linux server Instance
•	Login into Amazon LightSail using an Amazon Web Services account.
•	Once you are login into the site, click Create instance.
•	Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
•	Choose a instance plan (I took the cheapest, $5/month).
•	Keep the default name provided by AWS or rename your instance.(GraderApp)
•	Click the Create button to create the instance.
•	Wait for the instance to start up.
https://lightsail.aws.amazon.com/ls/webapp/us-east-1/instances/GraderApp/connect

# Get Instance connected to Remote Server: 

•	From the Account page for AWS LightSail instance : click SSH Keys
•	Download the Key
•	Save the key to your home drive LightsailPrivateKey.pem
•	Open your terminal  and cd into your home drive.  Then change permission to the downloaded key 
•	In terminal type chmod 600 LightsailDefaultKey.pem
•	Type:  mv LightsailDefaultKey.pem grader_key.rsa

# Check to see if connected  54.198.244.135 from my AWS instance
•	Run the following command on locally:   ssh -i grader_key.rsa ubuntu@54.198.244.135

It will show connected


To connect to EC2 instance you need the grader KEY:( supplied separately in the submit process)

ssh -i grader grader@54.198.244.135 -p 2200

# 1. Update the operating system packages and reboot if required
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade

# 2. Configure automatic security and critical updates Follow the official documentation:
Ubuntu Automatic Update Configuration

# 3. Set time zone to UTC
Check if the current time zone is set to UTC using:
$ date
Tue Jan 29 15:42:28 UTC 2019
If not UTC set time zone to UTC using the command below:
(select 'None of the above' from the menu and then select 'UTC'.)
$ sudo dpkg-reconfigure tzdata

Current default time zone: 'Etc/UTC'
Local time is now:      Tue Jan 29 15:45:18 UTC 2019.
Universal Time is now:  Tue Jan 29 15:45:18 UTC 2019.

# 4. Create user grader
$ sudo adduser grader
Adding user 'grader' ...
Adding new group 'grader' (1002) ...
Adding new user 'grader' (1002) with group 'grader' ...
Creating home directory '/home/grader' ...
Copying files from '/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
        Full Name []:  Grader
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y

# 5. Grant sudo permission to grader by editing /etc/sudoers.d directory
$ sudo touch /etc/sudoers.d/grader
$ sudo nano /etc/sudoers.d/grader

ADD:
Grader ALL=(ALL:ALL) NOPASSWD:ALL


# 6. Verify grader has sudo permission,  Create .SSH Directory and Keypairs for grader
$ su – grader 


CD/HOME/GRADER

$ mkdir .ssh
$ sudo touch .ssh/authorized_keys
$ sudo nano .ssh/authorized_keys ( add the public key contents of from grader.pub)



# 7. Create SSH Public Key  to place in .ssh/authorized_keys
OPEN LOCAL TERMINAL 

$ ssh-keygen
$ Generating public/private rsa key pair.
$ Enter file in which to save the key (/home/grader/.ssh/id_rsa): (Name it grader)
$ PassPhrase (leave blank or give name it grader)

THIS WILL GENERATE YOUR PUBLIC KEY -  THIS KEY IS WHAT IS PROVIDED TO GRADER
TO GET PUBLIC KEY
$ less grader.pub  (Copy contents and paste into .ssh/authorized_keys)

 CHECK TO SEE IF CONNECTS: on local terminal type:
$ ssh -i grader grader@54.198.244.135


# 8. Disable SSH logins through passwords in server permanently
$ sudo nano /etc/ssh/sshd_config

YOU HAVE UNCOMMENT EACH SECTION
- #What port, IPs and protocols we listen for:
Port 22   -  (change to Port 2200)

- #Authentication
PermitRootLogin yes   -  (change to NO)

- #Change to no to disable tunneled clear text passwords
PasswordAuthentication yes  - (change to NO)


$ sudo service sshd restart


CHECK YOU HAVE ACCESS  
$ ssh -i grader grader@54.198.244.135 -p 2200

# 9. Install Apache2 and enable required proxy modules and Install Python mod_wsgi
$ sudo apt-get install apache2
$ sudo a2enmod
give the below list of modules to enable

proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html

$sudo systemctl restart apache2


Install mod_wsgi
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install python-setuptools libapache2-mod-wsgi

START APACHE

$sudo service apache2 start

Check to see if working – go to any browser and type 54.198.244.135
 
# 10. Configure firewall to allow incoming to connect to Port 2200 port 80, and 123
$ sudo ufw app list
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH

$ sudo ufw default deny incoming
Rules updated
Rules updated (v6)

$ sudo ufw default deny outgoing
Rules updated
Rules updated (v6)

$ sudo ufw allow 80
Rules updated
Rules updated (v6)

$ sudo ufw allow 2200
Rules updated
Rules updated (v6)

$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

CHECK STATUS to ensure update 

$ sudo ufw status

$ sudo service ssh restart
$ sudo reboot

# 11. Enable port 2200 and HTTPS in the LightSail VM Networking settings
Open the LightSail Instance console. Go to Networking tab. Then in 'Firewall' settings
1.	Add a Custom TCP protocol with Port as 2200 to be enabled.
2.	Add HTTPS protocol to be enable

# 12. Install Pip and Virtualenv, create a virtual environment for web application and flask applications
 As grader
$ sudo apt-get install python3-pip
$ sudo apt-get install python-virtualenv

$ sudo virtualenv venv

$ source venv/bin/activate
$ sudo chmod 777 venv

# 13. Install PostgreSQL and configure and setup catalog2 database
 Install postgresql
$ sudo apt-get install postgresql
$ sudo apt-get install libpq-dev python
$ sudo apt-get install postgresql postgresql-contrib

-- Switch into postgresql superuser 'postgres'
$ sudo su - postgres

-- Enter psql shell
$ psql

-- Create user 'catalog2'
postgres=# CREATE ROLE catalog2 WITH LOGIN PASSWORD 'catalog2';
CREATE ROLE

-- Create database 'catalog2'
postgres=# CREATE DATABASE catalog2;
CREATE DATABASE

-- Grant all privileges on database 'catalog2' to user 'catalog2'
postgres=# REVOKE ALL ON SCHEMA public FROM public;
REVOKE
postgres=# GRANT ALL ON SCHEMA public TO catalog2;
GRANT
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog2 TO catalog2;
GRANT

-- Exit psql shell
postgres=# \q

-- Exit postgres user
$ exit

RESTART
$ sudo service apache2 restart
Use postgresql://catalog:catalog@localhost/catalog as database url in app.py, database_setup.py 

# 14. Install Git and Clone itemcatalog project
As grader
$ sudo apt-get install git

CREATE A FOLDER INSIDE /var/www/catalog2 and change into it
$ cd /var/www
$ sudo mkdir catalog2
$ sudo chown -R grader:grader catalog2
$ cd catalog2

CLONE YOUR PROJECT 
$ git clone https://github.com/mxor111/catalog2.git catalog2
(the project is not at /var/www/catalog2/catalog2

ONCE DONE – UPDATE THE database files  app.py / database_setup.py


CREATE A WSGI FILE:
$ sudo touch catalog2.wsgi
$ sudo nano catalog2.wsgi

ADD FOLLOWING CONTENT

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, “var/www/catalog2/catalog2/”)
sys.path.insert(1, “var/www/catalog2”)

from app import app as application
application.secret_key = ‘supersecretkey’

# 15. Install pip  and dependencies
$ sudo apt-get install python-pip

INSTALL DEPENDECIES
$ sudo pip install Flask
$ sudo pip install sqlalchemy
$ sudo pip install Flask-SQLAlchemy
$ sudo pip install psycopg2
$ sudo pip install flask-seasurf
$ sudo pip install oauth2client
$ sudo pip install httplib2
$ sudo pip install requests


ADD:
Grader ALL=(ALL:ALL) NOPASSWD:ALL

# 16. Configure and Enable Virtual environment
$ sudo nano /etc/apache2/sites-available/catalog2.conf

ADD THE FOLLOWING CONTENT:

<VirtualHost *.80>
    ServerName 54.198.244.135
    ServerAlias www.54.198.244.135.xip.io
    Service Admin mishtay@yahoo.com
WSGISCRIPTALIAS / /var/www/catalog2/catalog2.wsgi
<Directory /var/www/catalog2/>
    WSGIProcessGroup application
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
</Directory>
   Alias/static/var/www/catalog2/static
<Directory /var/www/catalog2/static/>
Order allow,deny
Allow from all
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
CustomLog ${APACHE_LOG_DIR}access.log combined
</VirtualHost>


CHECK FOR SYNTEX ERRORS
$ sudo apache2ctl configtest
Syntax OK


 ENABLE THE VIRTUAL HOST
$ sudo a2ensite catalog2

RESTART
$ sudo service apache2 restart


# 17. Edit Client Secrets - Connect to your project in Google API credentials and add the IP address as Authorized redirect URIs and Authorized JavaScript origins.
Note: When you set up OAuth for your application, you will need a DNS name that refers to your instance's IP address. You can use the xip.io service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name 54.92.179.141.xip.io refers to the server above
UPDATE CLIENT SECRETS:

$ cd /var/www/catalog2/catalog2
$ sudo nano client_secrets.json
update credentials

REPLACE THE CLIENT ID IN login.html
$ cd /var/www/catalog2/catalog2/templates/login.html

UPDATE GOOGLE app.py
$oauth_flow = flow_from_clientsecrets(‘/var/www/catalog2/catalog2/client_secrets.json’, scope=”)


Google API will not accept Public IPs .  You need to have a domain name -  Once you got your domain name add 54.198.244.135.xip.io .tp Virtual Host
Authenticate Login through Google:
•	Go to Google Cloud .
•	Click APIs & services on left menu.
•	Click Credentials.
•	Create an OAuth Client ID (under the Credentials tab), and add http://54.198.244.135 and http://ec2-54-198.244.135.us-east-1.compute.amazonaws.com as authorized JavaScript origins.
•	Add http://ec2-54.198.244.135.us-east-1.compute.amazonaws.com/oauth2callback as authorized redirect URI.
•	Download the corresponding JSON file, open it et copy the contents.
•	Open /var/www/catalog2/catalog2/client_secret.json and paste the previous contents into the this file.
 
 # 18.Update Application to accommodate change: 

UPDATE app.py
$ sudo nano app.py

CLIENT_ID = json.load(open(‘/var/www/catalog2/client_secrets.json’. ‘r’ .read())[‘web’][‘client_id’]

engine = create_engine(‘postgresql://var/www/catalog2/catalog2/itemscatalog.db’)

UPDATE FACEBOOK
app_id = json.loads(open(‘/var/www/catalog2/catalog2/fb_client_secrets.json’, ‘r’ .read())[‘web’][‘app_id’]

app_secret = json.loads(open(‘/var/www/catalog2/catalog2/fb_client_secrets.json’, ‘r’ .read())[‘web’][]app_secret’]

UPDATE Database-setup.py
$sudo nano database_setup.py

engine = create_engine(‘postgresql://var/www/catalog2/catalog2/itemscatalog.db’)

UPDATE FACEBOOK


# 19. Automatically Install Updates The unattended-upgrades package can be used to automatically install important system updates. Enable upgrades:

sudo apt-get install unattended-upgrades

sudo dpkg-reconfigure --priority-low unattended-upgrades

sudo service apache2 restart

# 20. Additional Security Add feature to protect against attackers, I installed fail2ban a software package that blocks up address with multiple failed login attempts within a certain about of time 
$sudo apt-get update
$sudo apt-get install fail2ban

# copy default configuration file to new file to prevent updates from overwriting config settings.  

$ sudo cp /etc/fail2ban/jail.conf  /etc/fail2ban/jail.local
$ sudo nano /etc/fail2ban/jail.conf
# change port to 2200

$sudo service fail2ban restart


# 21. DEBUGGING 
sudo tail -f /var/log/apache2/error.log

sudo nano /var/log/apache2/error.log

sudo apache2ctl configtest


#RESOURCES: 
Official Ubuntu Documentation, Automatic Updates.
DigitalOcean, How To Protect SSH with Fail2Ban on Ubuntu 14.04.
Fail2Ban Official website
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html
https://www.digitalocean.com/docs/droplets/resources/troubleshooting-ssh/connectivity/
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
https://help.ubuntu.com/community/PostgreSQL
http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442
DigitalOcean, How To Set Up SSH Keys.         Ubuntu Wiki, SSH/OpenSSH/Keys
DigitalOcean, How To Add and Delete Users on an Ubuntu 14.04 VPS
