Linux Server Configuration Project 
this is last project for Udacity FSND , this project after strugling on catalog item project and creating 
books catalog with categories & using CRUD all of this was using localhost know we should deploy
our work into a real world .
in this project we are using amazon lightsail 

General information :
ip address : 18.194.41.143
URL : ec2-18-194-41-143.eu-central-1.compute.amazonaws.com

creating a new instance :
1- go to https://lightsail.aws.amazon.com
2- create a new account
3- click create instance
4- click linux & os only
5- Chose any Ubuntu Version you like 
6- name your instance
7- create 

connecting to the server :
1- go to your instance
2- go to your account page
3- go to SSH keys 
4- download the key 
5- cd to downloads 
6- chmod 600 YourAWSKey.pem
7-  ssh -i YourAWSKey.pem ubuntu@18.194.41.143


server configuration :

after connecting you should switch to root user 
sudo su -

then create a new user called "grader" 
sudo adduser grader

give a sudo permission to grader user :
sudo nano /etc/sudoers.d/grader
type inside :
grader ALL=(ALL:ALL) ALL
save it by pressing ctrl + x  then type y and enter to save changes


after creating the user now we want to connect to that user by using pair keys
on your local machine's terminal type :
ssh-keygen -f ~/.ssh/udacity_key.rsa
this command will generate a public and private key for us 
we gonna use that to connect between our local machine and the server 
the private key will be on our computer and public key on the server
 open public key file :
 cat ~/.ssh/udacity_key.rsa.pub
 copy the public key 

 get back to your server's terminal that you connected to by using ssh -i YourAWSKey.pem ubuntu@18.194.41.143
 cd to grader user :
 cd /home/grader
 Create a .ssh directory: 
  mkdir .ssh

Create a file to store the public key: 
touch .ssh/authorized_keys

Edit the authorized_keys file :
nano .ssh/authorized_keys

Change the permission: 
sudo chmod 700 /home/grader/.ssh and $ sudo chmod 644 /home/grader/.ssh/authorized_keys

Change the owner from root to grader: 
sudo chown -R grader:grader /home/grader/.ssh

Restart the ssh service: 
sudo service ssh restart



login to the server by using grader user :
ssh -i ~/.ssh/udacity_key.rsa grader@18.194.41.143
make sure that you're up to date:
sudo apt-get update
sudo apt-get upgrade

now we shoud prevent user from using a password :
sudo nano /etc/ssh/sshd_config 
PasswordAuthentication line and change text after to no. After this
We now need to change the ssh port from 22 to 2200 
sudo nano /etc/ssh/ssdh_config Find the Port line and change 22 to 2200. 
 restart ssh: 
 sudo service ssh restart
disconnect from your server :
 ~.
 connect again with new port 2200 :
 ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.194.41.143 
 Disable ssh login for root user: 
 sudo nano /etc/ssh/sshd_config Find the PermitRootLogin line and edit to no
 Restart ssh :
 sudo service ssh restart

 Now we need to configure UFW to fulfill the requirement:

sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable



now we're done with security part it's time to deploy your application 

first install appache2 and required packages :
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo apt-get install git

Enable mod_wsgi by :
sudo a2enmod wsgi 
and start the web server by :
sudo service apache2 start 

Set up the folder structure :
cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog
Now we clone the project from Github: $ git clone [your link] catalog Copy your link from here is the easiet way: 


Create a .wsgi file: 
sudo nano catalog.wsgi 
and add the following into this file :
#beiging
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey' 
#end

Rename the application.py to __init__.py

Now we need to install and start the virtual machine

sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv



Now we need to install the Flask and other packages needed for this application
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install "anything else you have built within this application"

edit json path in your python file to 
/var/www/catalog/catalog/client_secrets.json 

edit your json file 
instaed of local host to your domain

Now we need to configure and enable the virtual host:
sudo nano /etc/apache2/sites-available/catalog.conf
Paste the following code and save :

#beiging 
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
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
</VirtualHost>

#end

Now we need to set up the database
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres -i

Now we create a user to create and set up the database. I name my database catalog with user catalog
CREATE USER catalog WITH PASSWORD [your password];
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
Connect to database $ \c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
Quit the postgrel command line: $ \c and then $ exit

use sudo nano command to change all engine to :
engine = create_engine('postgresql://catalog:[your password]@localhost/catalog 

excute your database_setup file & seeder if you have it


Restart Apache server :
sudo service apache2 restart and enter your public IP address or host name into the browser. Hooray! Your application should be online now!


Reference:
https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md
https://github.com/stueken/FSND-P5_Linux-Server-Configuration
https://github.com/iliketomatoes/linux_server_configuration