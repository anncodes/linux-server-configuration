# Linux Server Configuration Project
## Udacity Full Stack Web Developer Nanodegree
Deploying to Linux Servers Module

### About the project
Baseline installation of a Linux server and prepare it to host web applications.  Server must be secured from a number of attack vectors.  A database server will be installed and configured.  Existing web applications will then be deployed onto it.

- IP address : 35.180.79.12
- SSH port : 2200
- Application URL : [http://35.180.79.12.xip.io](http://35.180.79.12.xip.io)
- Deployed application : [Item Catalog App](https://github.com/anncodes/catalog-app)

### Get your server
The project requires [Amazon Lightsail](https://aws.amazon.com/) to create a Linux server instance.
- Create a new Ubuntu Linux server instance on Amazon Lightsail.
1. Log in.
2. Create an instance and choose an Ubuntu instance image.
   - First, choose "OS Only".Second, choose "Ubuntu" as the operating System.
3. Choose an instance plan and give your instance a hostname.
4. Once your instance has started, log into it with SSH.  Configure the Lightsail firewall.

### Configure the server
1. Update all currently installed packages.
   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```
2. Change the SSH port from 22 to 2200. Configure the Lightsail firewall to allow it.
```
sudo nano /etc/ssh/sshd_config
```
Change port from 22 to 2200

3. Configure the UFW to allow incoming connections for SSH(port 2200), HTTP(port 80), and NTP(port123).
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

> WARNING: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

### Give `grader` access.
1. Create a new user account named `grader`.
```
sudo adduser grader
```
2. Give `grader`the permission to `sudo`.
```
sudo nano /etc/sudoers.d/grader
```
Add text `grader ALL=(ALL) NOPASSWD:ALL`
3. Create an SSH key pair for `grader` using `ssh-keygen` tool.
```
ssh-keygen -t rsa
```
   - Save the private key in `~/.ssh` on local machine.
   - Deploy public key on development environment. On virtual machine:
    ```
    su - grader
    mkdir .ssh
    cd .ssh
    sudo nano authorized_keys
    ```
    Copy the public key generated on your local machine to this file and save.
    ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    ```
4. Reload `SSH`
5. Login `ssh -i ~/.ssh/udacity_key grader@35.180.79.12 -p 2200`

### Prepare to deploy the project
1. Configure the local timezone to UTC.
```
sudo dpkg-reconfigure tzdata
```
Select `none of the above` then `UTC`.
2. Install and configure Apache to serve PYthon mod_wsgi application.
```
sudo apt-get install apache2
```
3. Install mod_wsgi.
```
sudo apt-get install libapache2-mod-wsgi python-dev
```
4. Restart Apache `sudo service apache2 restart`.

### Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`.
2. Do not allow remote connections `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`.
3. Login as user 'postgres' `sudo su - postgres`.
4. `psql`
5. Create new database named catalog and new user 'catalog' in PostgreSQL shell.
```
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
### Deploy the Item Catalog project
1. Install Git `sudo apt-get install git`.
2. `cd /var/www`
3. `sudo mkdir catalog`
4. `cd /catalog`
5. Clone project from github `git clone https://github.com/anncodes/catalog-app.git catalog`
6. Get in inner catalog directory `cd catalog`
7. Rename application.py to __init__.py `sudo mv application.py __init__.py`.
8. Modify database_setup.py, __init__.py and catalogdata.py
   - Change engine line to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Configure and enable a new virtual host
1. Install pip `sudo apt-get install python-pip`
2. Install virtual environment
   - Install virtual environment `sudo pip install virtualenv`
   - Create a new virtual environment `sudo virtualenv venv`
   - Activate the virtual environment `source venv/bin/activate`
   - Change permissions `sudo chmod -R 777 venv`
3. Install Flask and other dependencies
    - Install Flask `pip install Flask`
    - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`
3. Update path of client_secrets.json file
    - `nano __init__.py`
    - Change the path to `/var/www/catalog/catalog/client_secrets.json`
4. Create a new catalog.conf file `sudo nano /etc/apache2/sites-available/catalog.conf` and paste the following code below:
```
<VirtualHost *:80>
	ServerName 35.180.79.12
	ServerAlias 35.180.79.12.xip.io
	ServerAdmin agepxxx@gmail.com
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
```
5. Enable the virtual host `sudo a2ensite catalog`.

### Create the .wsgi file
1. Run `sudo nano /var/www/catalog/catalog.wsgi`
   - Paste the following code on it:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'megasecretkey'
```

### Restart Apache 
`sudo service apache2 restart`

### Site
Visit the Catalog App at http://35.180.79.12 or http://35.180.79.12.xip.io

## Resources
https://tutorials.ubuntu.com/tutorial/install-and-configure-apache#3
Dealing with the psycopg2-binary warning: https://github.com/psycopg/psycopg2/issues/674
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-four-%E2%80%93-configure-and-enable-a-new-virtual-host
