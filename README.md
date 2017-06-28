# Linux Server Configuration

## About

In this project, a baseline installation of a Linux server was given and it was to be prepared to host a web applications. Server was also to be made secure from a number of attack vectors. installing and configuring a database server, and deploying one existing web applications onto it was to be done.

## Server Details

 1. IP: 13.126.206.104, SSH port: 2200
 2. URL of hosted application: ec2-13-126-206-104.ap-south-1.compute.amazonaws.com

## List of Software/Pakages Installed
 
 1. sqlalchemy
 2. apache2
 3. postgresql
 4. postgresql-contrib
 5. psycopg2
 6. libapache2-mod-wsgi
 7. Flask
 8. oauth2client
 9. httplib2
 10. requests
 11. virtualenv
 12. python-pip
 13. python-psycopg2
 14. python-dev

## Tasks

The project is divided into a number of different tasks. Below is the list of tasks with my solution.

### Update all currently installed packages

 1. Run `sudo apt-get update` to get list of all available update.
 2. Run `sudo apt-get upgrade` to update all packages for whom update is available.

### Change the SSH port from 22 to 2200

 1. Run `sudo nano /etc/ssh/sshd_config` 
 2. Find *PORT 22*(at around line# 5-6) and change it to *PORT 2200*.
 3. Run `sudo service ssh restart` for restarting ssh.


### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

 1. Run `sudo ufw status` for checking the current status of *Uncomplicated Firewall*. If Firewall was not configured before then it will show *Status: inactive
*.
 2. Run `sudo ufw default deny incoming` for blocking all incoming ports.
 3. Run `sudo ufw default allow outgoing` for allowing all outgoing ports.
 4. Run `sudo ufw allow 2200/tcp` for allowing incoming connection for *SSH Port*.
 5. Run `sudo ufw allow www` for allowing incoming connection for *HTTP Port*.
 6. Run `sudo ufw allow 123/udp` for allowing incoming connection for *NTP Port*
 7. Run `sudo ufw enable` for enabling *Uncomplicated Firewall*.
 8. Run `sudo ufw status` to check status of Firewall. It should list all allowed incoming ports.
 9. Back on *Lightsail instance page* select *Networking Tab* and click *Edit Rules* under *Firewall* Section.
 10. Add Custom Port *2200/TCP*.
 11. Add Custom Port *123/UDP*.
 12. Remove *SSH Port 22*.
 13. After this open a new one this time using port 2200.

### Create a new user account named grader

 1. Run `sudo adduser grader` for creating new user.
 2. Enter password twice.
 3. Enter desciption and confirm.
 
### Give grader the permission to sudo

 1. Run `sudo ls /etc/sudoers.d` for listing contents of *sudoers.d directory*.
 2. A configuration and README file will be present.
 3. Run `sudo cp /etc/sudoers.d/name_of_configuration_file /etc/sudoers.d/grader`
 4. Run `sudo ls /etc/sudoers.d` again. This time there will be a new grader file.
 5. Run `sudo nano /etc/sudoers.d/grader`.
 6. Edit *ubuntu* from line#4 to *grader*.
 7. Save the file.

### Create and install an SSH key pair for grader using the ssh-keygen tool

 1. On a new terminal open your local vagrant machine and run `ssh-keygen`.
 2. Enter */home/vagrant/.ssh/your_filename*
 3. Enter passphrase two times. This will create two files on your local machine- *your_filename* and *your_filename.pub*.
 4. Run `cat .ssh/your_filename.pub`. Copy the contents shown.
 5. Back on your server terminal, run `sudo -i -u grader` to switch to grader user.
 6. Run `mkdir .ssh` and then run `touch .ssh/authorized_keys`.
 7. Run `nano .ssh/authorized_keys` and paste the contents copied from your_filename.pub file from your local vagrant machine.
 8. Save the file.

### Configure the local timezone to UTC

 1. Run `sudo dpkg-reconfigure tzdata` and follow the gui on screen.
 2. Select *None of the above*.
 3. Then select **UTC**.

### Install and configure Apache to serve a Python mod_wsgi application
 
 1. Run `sudo apt-get install apache2 libapache2-mod-wsgi python-dev` for installing *Apache2* and *mod_wsgi*.
 2. Run `sudo a2enmod wsgi` for enabling *mod_wsgi*.
 3. Run `cd /var/www` then `sudo mkdir FlaskApp`.
 4. Run `cd FlaskApp` and again run `sudo mkdir FlaskApp`.
 5. Run `cd FlaskApp` and `sudo mkdir static templates`. Steps 3-5 are done to provide a directory structure to the project.
 6. Run `sudo nano __init__.py` for creating a new python file.
 7. Add the following contents to it: 
```    
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello World!"
if __name__ == "__main__":
    app.run() 
```
 8. Run `sudo apt-get install python-pip`. We will use *pip* for installing other packages.
 9. Run `sudo pip install virtualenv`.
 10. Run `sudo virtualenv virtualenv_name` and then run `source virtualenv_name/bin/activate` for starting a virtual environment.
 11. Run `sudo pip install Flask sqlalchemy oauth2client httplib2 requests` for installing flask and other packages.
 12. Run `sudo python __init__.py` for testing the apache server. A message with *Running on http://localhost:5000/* will be shown.
 13. Run `deactivate` to deactivate the virtual environment.
 14. In a new terminal run `nslookup SERVER_PUBLIC_IP`, copy the dns name and exit the terminal.
 14. Run `sudo nano /etc/apache2/sites-available/FlaskApp.conf` and paste the below contents in it after replacing the *dns name* and &email*.
```
<VirtualHost *:80>
		ServerName YOUR_DNS_NAME
		ServerAdmin your_email@address.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
 15. Run `sudo a2ensite FlaskApp` to enable the visual host.
 16. Run `cd /var/www/FlaskApp` and `sudo nano flaskapp.wsgi`.
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```
 17. Run `sudo service apache2 restart` for restarting the *apache2* server.
 18. Open Web browser and enter your dns name from above. You will be able to see **Hello World!**.

### Install and configure PostgreSQL

 1. Open another instance of terminal and connect it to the server.
 2. Run `sudo apt-get install postgresql postgresql-contrib` for installing *Postgresql* and *-contrib* package.
 3. Run `sudo -i -u postgres` for switching to postgres superuser.
 4. Run `createuser --interactive` for creating a new role.
 5. Enter the name of the role- *catalog* and then select **no** for *Superuser*, **yes** for *ability to create new database* and the lastly **no** for ability to create new role.
 6. Run `createdb catalog` for creating a new database.
 7. Logout from the postgresql account and run `sudo adduser catalog`.
 8. Enter password two times and then enter description.
 9. Run `sudo -i -u catalog` and then `psql`.
 10. Run `\conninfo` for seeing connection information.
 11. Run `\password catalog` and enter password twice.
 12. Run `\du` for seeing permissions.

### Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program

 1. Back on terminal with grade user logged in.
 2. From grader directory run `cd /var/www/FlaskApp/FlaskApp`.
 3. Run `sudo git clone https://github.com/ravikumarseth/catalog.git`.
 4. Run `ls` to see the structure of the directories.
 5. Run `cd catalog` then run `sudo mv * ./..` and `cd ..`.
 6. Run `sudo rm -rf catalog README.md  games.db` to remove unncessary files.
 7. Run `sudo apt-get install python-psycopg2` for installing *psycopg2*.

### Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

 1. Run `sudo nano gamegenre.py`, `sudo nano database_setup.py` and `sudo nano __init__.py` and change the line `sqlite:///games.db` to `postgresql://catalog:password@localhost/catalog`
 2. In **__init__.py** edit the part at the end of the file so that it looks like this:
```
if __name__ == '__main__':
    app.run()
```
 3. Update *client_secrets.json* with the updated one created after updating *Authorized JavaScript origins* and *Authorized redirect URIs* using *google developer console*.
 
 4. Run `sudo service apache2 restart` for restarting the *apache2* server.
 
 5. If any error pop-up then run `sudo cat /var/log/apache2/error.log` and read through error log to find solution.
 
## References

 1. [UbuntuTime- Community Wiki Help](https://help.ubuntu.com/community/UbuntuTime)
 2. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 3. [How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
 4. [Configuration -- Flask-SQLAlchemy Documentation(2.1)](http://flask-sqlalchemy.pocoo.org/2.1/config/)
 5. [Stackoverflow Answer](https://stackoverflow.com/a/6438520/6032818)
