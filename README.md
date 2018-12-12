# Project Linux Server Configuration (Udacity)

Linux Server Configuration project of Udacity FullStack Nanodegree

- The IP address of the project is : `35.227.44.250`
- Accessible Port is `2200`
- The Project can be accessed by visiting the URL :  ` http://35.227.44.250.xip.io/`



### Project Overview

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.


### Why This Project?

A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

### What Will I Learn?
> You will learn how to access, secure, and perform the initial configuration of a bare-bones 
> Linux server. You will then learn how to install and configure a web and database server and 
> actually host a web application.


### Get your server

1. Start a new Ubuntu Linux server instance on Google Cloud Platform. 
- Create an instance in `Computer Engine` of Google Cloud.
- Choose an instance image: Ubuntu
2. SSH into your server.
 - After the server has been created, click on `SSH` button 
 - Copy the SSH key and place it in `~/.ssh` directory , name it `id_rsa`
 - Go to `Metadata` on google cloud platform, select `SSH keys`, `Edit` and add the `id_rsa.pub`, name it `aysait101`.
 - Type `ssh -i ~/.ssh/id_rsa aysait101@35.227.44.250` to SSH from your terminal, enter passphrase to connect.

### Secure your server
3. Update all currently installed packages.
- `sudo apt-get update`
- `sudo apt-get upgrade`
4. Change the SSH port from 22 to 2200. 
- Go to `VM instance` interface, `View network details`, navigate to `Firewall rules` and add custom port `tcp:2200` to allow connection to the port.
- While logged in from terminal, `sudo nano /etc/ssh/sshd_config` and change `Port 22` to `Port 2200` and change `PasswordAuthentication no` to enfirce SSH key login.
- Restart with `sudo service ssh restart`
- Login using `ssh -i ~/.ssh/id_rsa -p 2200 grader@35.227.44.250` and enter passphrase to connect.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
- `sudo apt-get install ufw` to install ufw
- `sudo ufw allow 2200/tcp`
- `sudo ufw allow 80/tcp`
- `sudo ufw allow 123/udp`
- `sudo ufw enable`
- check status using `sudo ufw status`


### Give ```grader``` access.
In order for your project to be reviewed, the grader needs to be able to log in to your server.

6. Create a new user account named `grader`.
- Logged in as root, `sudo adduser grader`
- Enter the details to create user.
7. Give `grader` the permission to `sudo`.
- `sudo nano /etc/sudoers`
- `touch /etc/sudoers.d/grader`
- `sudo nano /etc/sudoers.d/grader`, enter the following: `grader ALL=(ALL:ALL) ALL`and save.
8. Create an SSH key pair for `grader` using the `ssh-keygen` tool.
- On your machine, generate SSH key pair using `ssh-keygen` in `~/.ssh`. (I name it linuxTwo)
- `su - grader` logs you in as grader using password.
- `mkdir .ssh`
- `touch .ssh/authorized_keys`
- `sudo nano .ssh/authorized_keys` and paste the contents of `linuxTwo.pub`.
- `chmod 700 .ssh`, `chmod 644 .ssh/authorized_keys`
- `service ssh restart` to restart SSH and `CTRL D` to logout
- Use `ssh -i ~/.ssh/linuxTwo grader@35.227.44.250 -p 2200` to login using passphrase.

### Prepare to deploy your project.
9. Configure the local timezone to UTC.
- `sudo dpkg-reconfigure tzdata` and choose your region.
10. Install and configure Apache to serve a Python mod_wsgi application.
- `sudo apt-get install apache2`
- `sudo apt-get install libapache2-mod-wsgi python-devsudo apt-get install libapache2-mod-wsgi pytdev`
- `sudo apt-get build-dep python-psycopg2`.

11. Install and configure `PostgreSQL:`
- `sudo apt-get install postgresql postgresql-contrib`
- Create a new database user named `catalog` that has limited permissions to your catalog application database.
- `sudo su - postgres`
- `CREATE USER catalog WITH PASSWORD 'PASSWORD';`
- `ALTER USER catalog CREATEDB;`
- `CREATE DATABASE catalog WITH OWNER catalog;`
- `GRANT ALL ON SCHEMA public TO catalog;`
- `\q`
12. Install `git`.
- `sudo apt-get install git`

### Deploy the Item Catalog project.
13. Clone and setup your `Item Catalog` project from the Github repository you created earlier in this Nanodegree program.
- `cd /var/www`
- `sudo mkdir catalog`
- `cd /catalog`
- `touch catalog.wsgi` to create .wsgi file
- `sudo nano catalog.wsgi` and add the folowing:
 ```
 #!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
 ```
- clone project from `git clone https://github.com/ayseth/item-catalog.git`
- Renname folder `mv item-catalog catalog`
- `cd /catalog`
-  Rename `application.py` using `mv application.py __init__.py`
-  `sudo nano __init__.py`, go to the end of the file and change 
```
#app.debug = True
#app.run(host='0.0.0.0', port=5000)
app.run()
```
- and change `engine = create_engine('sqlite:///catalog.db?check_same_thread=False')` to `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
- `sudo nano database_setup.py` and change `engine = create_engine('sqlite:///catalog.db?check_same_thread=False')` to `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`
- `sudo python database_setup.py`
- `sudo apt-get install virtualenv`
- `sudo virtualenv venv`
- `source venv/bin/activate` to activate virtual environment
- `sudo chmod -R 777 venv`
- `sudo apt-get install python-pip`
- Install the following,
  - `pip install sqlalchemy`
  - `pip install httplib2`
  - `pip install requests`
  - `pip install flask`
  - `pip install oauth2client`
  - `pip install Flask-SQLAlchemy`
  - `pip install psycopg2`
  - `pip install psycopg2-binary`
- `deactive` to deactivate virtual env
- `sudo nano /etc/apache2/sites-available/catalog.conf` and add the following 
```
<VirtualHost *:80>
    ServerName 35.227.44.250.xip.io
    ServerAdmin aysait101@35.227.44.250
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/ven$
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
```
- `sudo a2ensite catalog`
- `service apache2 reload`
- `sudo service apache2 restart`

14. To check for errors use `sudo tail /var/log/apache2/error.log`
## Other important changes:
- For google login to work, add go to google cloud credentials and add `http://35.227.44.250.xip.io` to `Authorized JavaScript origins`
- Add `	http://35.227.44.250.xip.io/gconnect`, `	http://35.227.44.250.xip.io/gdisconnect`, `http://35.227.44.250.xip.io/login` to `Authorized redirect URIs`
- Download the `client_secerts.json`
- On your terminal replace the contnets of the old `client_secrets.json` file with the new `client_secrets.json` using `sudo nano /var/www/catalog/catalog/client_secrets.json` and save.
- Change the path of `client_secrets.json` in `__init__.py`

### References:
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://www.digitalocean.com/community/questions/flask-and-wsgi-importerror-cannot-import-name-app
- https://medium.com/@Riverside/how-to-install-apache-php-postgresql-lapp-on-ubuntu-16-04-adb00042c45d
- https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e
- https://www.ostechnix.com/configure-apache-virtual-hosts-ubuntu-part-1/
- https://stackoverflow.com/questions/53586358/apache-mod-wsgi-google-oauth2-0-error-400-invalid-request
- http://exploreflask.com/en/latest/configuration.html



