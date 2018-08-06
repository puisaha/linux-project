

public Ip: 18.191.233.252
SSH PORT: 2200

###For starting a new Ubuntu Linux server instance on Amazon Lightsail.
Launch Amazon Lightsail terminal

Must be logged into your Amazon Web Services account.
Visit this link and press Create new instance of Ubuntu.
You will get your respective public IP address.
Download the default key-pair and copy to /.ssh folder.
Open your terminal and type in chmod 600 ~/.ssh/key.pem
Now Use the command ssh -i ~/.ssh/key.pem ubuntu@18.191.233.252 to create the instance on your terminal

###For creating a grader user :

sudo adduser grader

###Give the grader the permission to sudo:

sudo visudo
inside the file add grader ALL=(ALL:ALL) ALL below the root user under "#User privilege specification"
save file(nano: ctrl+x, Y, Enter)
Add grader to /etc/sudoers.d/ and type in grader ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/grader
Add root to /etc/sudoers.d/ and type in root ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/root

###Update all currently installed packages:

Find updates:sudo apt-get update
Install updates:sudo sudo apt-get upgrade Hit Y for yes and give yourself a break while it installs.

###Change the SSH port from 22 to 2200: 

vi /etc/ssh/sshd_config add port 2200 below port 22
while in the file also change PermitRootLogin prohibit-password to PermitRootLogin no to disallow root login
Change PasswordAuthentication from no to yes. We will change back after finishing SHH login setup
save file
restart ssh servicesudo service ssh reload

###Create SSH keys and copy to server manually:

On your local machine generate SSH key pair with: ssh-keygen
save youkeygen file in your ssh directory /home/ubuntu/.ssh/ example full file path that could be used: /home/ubuntu/.ssh/catalog
You can add a password to use encase your keygen file gets compromised(you will be prompted to enter this password when you connect with key pair)
Change the SSH port number configuration in Amazon lightsail in networking tab to 2200.
login into grader account using password set during user creation ssh -v grader@*Public-IP-Address* -p 2200
Make .ssh directorymkdir .ssh
make file to store keytouch .ssh/authorized_keys
On your local machine read contents of the public key cat .ssh/project5.pub
Copy the key and paste in the file you just created in grader nano .ssh/authorized_keys paste contents(ctr+v)
save file
Set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys
Change PasswordAuthentication from yes back to no. nano /etc/ssh/sshd_config
save file
login with key pair: ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/project5

###Configure the local timezone to UTC :

run sudo dpkg-reconfigure tzdata from prompt: select none of the above. Then select UTC.

###Install and configure Apache to serve a Python mod_wsgi application :

sudo apt-get install apache2 Check if "It works!" at you public IP address given during setup.
install mod_wsgi: sudo apt-get install libapache2-mod-wsgi
configure Apache to handle requests using the WSGI module sudo nano /etc/apache2/sites-enabled/000-default.conf
add WSGIScriptAlias / /var/www/html/myapp.wsgi before </VirtualHost> closing line
save file(nano: ctrl+x, Y, Enter)
Restart Apache sudo apache2ctl restart

###Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

install git

sudo apt-get install git
install python dev and verify WSGI is enabled

Install python-dev packagesudo apt-get install python-dev
Verify wsgi is enabled sudo a2enmod wsgi

###Create flask app :

cd /var/www
sudo mkdir catalog
cd catalog
sudo mkdir catalog
cd catalog
sudo mkdir static templates
sudo nano __init__.py
 from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, world (Testing!)"
if __name__ == "__main__":
app.run()

###install flask

sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
sudo chmod -R 777 venv
source venv/bin/activate
pip install Flask
python __init__.py
deactivate

###Configure And Enable New Virtual Host

Create host config file sudo nano /etc/apache2/sites-available/catalog.conf
paste the following:
<VirtualHost *:80>
  ServerName 34.201.114.178
  ServerAdmin admin@34.201.114.178
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
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Enable `sudo a2ensite catalog`

* Create the wsgi file
    * `cd /var/www/catalog`
    * `sudo nano catalog.wsgi`

    ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
  
  ####save file

sudo service apache2 restart

Clone Github Repo

sudo git clone https://github.com/
make sure you get hidden files iin move shopt -s dotglob. Move files from clone directory to catalog mv /var/www/catalog/item-catalog/* /var/www/catalog/catalog/
remove clone directory sudo rm -r devpost
make .git inaccessible

from cd /var/www/catalog/ create .htaccess file sudo nano .htaccess
paste in RedirectMatch 404 /\.git
save file

####
install dependencies:

source venv/bin/activate
pip install httplib2
pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
pip install Flask-SQLAlchemy
sudo pip install python-psycopg2

####
Install and configure PostgreSQL:

Install postgressudo apt-get install postgresql
install additional modelssudo apt-get install postgresql-contrib
by default no remote connections are not allowed
config database_setup.py sudo nano database_setup.py
python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')
repeat for project.py
copy your main project.py file into the init.py file mv project.py __init__.py
Add catalog user sudo adduser catalog
login as postgres super usersudo su - postgres
enter postgrespsql
Create user catalogCREATE USER catalog WITH PASSWORD 'db-password';
Change role of user catalog to creatDBALTER USER catalog CREATEDB;
List all users and roles to verify\du
Create new DB "catalog" with own of catalogCREATE DATABASE catalog WITH OWNER catalog;
Connect to database\c catalog
Revoke all rights REVOKE ALL ON SCHEMA public FROM public;
Give accessto only catalog roleGRANT ALL ON SCHEMA public TO catalog;
Quit postgres\q
logout from postgres super userexit
Setup your database schema python database_setup.py

#####
fix OAuth to work with hosted Application

Google wont allow the IP address to make redirects so we need to set up the host name address to be usable.
go to http://www.hcidata.info/host2ip.cgi to get your host name by entering your public IP address Udacity gave you.
open apache configbfile sudo nano /etc/apache2/sites-available/catalog.conf
below the ServerAdmin paste ServerAlias YOURHOSTNAME
make sure the virtual host is enabled sudo a2ensite catalog
restart apache server sudo service apache2 restart


