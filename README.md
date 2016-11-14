# Linux Server Configuration

## Project Overview:

> You will take a baseline installation of a Linux distribution on a virtual machine
> and prepare it to host your web applications, to include installing updates,
> securing it from a number of attack vectors and installing/configuring web and database
> servers.

The application meant to be deployed is the **Item catalog app**, previously developed
for [Project 3](https://github.com/iliketomatoes/catalog).

## Server Details

* IP address: 35.160.12.62
* SSH port: 2200
* URL: http://ec2-35-160-12-62.us-west-2.compute.amazonaws.com/

## Configuration Steps

### Development Environment Information

1. Download Private Key (udacity_key.rsa)
2. Change permission to read write for owner and none for group and everyone ``` chmod 600 ~/.ssh/udacity_key.rsa ```
3. Connect to the machine using ``` ssh ``` as root ``` ssh -i ~/.ssh/udacity_key.rsa root@{YOUR.IP.GOES.HERE} ```


### User

1. Create a new user named grader ``` adduser grader ```
2. Create a new file under the suoders directory: ``` nano /etc/sudoers.d/grader ```
3. Grand the user the sudo permission  by adding the following line in the created file
``` grader ALL=(ALL:ALL) ALL ```
4. To prevent the "sudo: unable to resolve host" error (when connecting as **grader**), edit the hosts:

	* ``` nano /etc/hosts ```
	* Add the host: 127.0.1.1 localhost ip-10-20-53-117

#### Set-up SSH Keys For User **grader**

1. Create ssh folder under **grader** home directory:
``` mkdir /home/grader/.ssh ```

2. Grant the user **grader** the ownership of the ssh folder:
``` chown grader:grader /home/grader/.ssh ```

3. Grant the user **grader** the ability to read write and execute the ssh folder:
``` chmod 700 /home/grader/.ssh ```

4. Copy authorized keys:
``` cp /root/.ssh/authorized_keys /home/grader/.ssh/ ```

5. Grant the user **grader** the ownership of the key file:
``` chown grader:grader /home/grader/.ssh/authorized_keys ```

6. Grant the user **grader** the ability to read write the ssh folder:
``` chmod 644 /home/grader/.ssh/authorized_keys ```

#### Disable **root** Login

1. Navigate to ``` /etc/ssh/sshd_config ```
2. Change PermitRootLogin From without-password to no
3. Restart the SSH service: ``` service ssh restart ```

### Change SSH Port

1. Navigate to ``` /etc/ssh/sshd_config ```
2. Change Port from 22 to  2200
3. Restart the SSH service: ``` service ssh restart ```

### Change Timezone to UTC

1. Change timezone to UTC: ``` timedatectl set-timezone UTC ```
2. Check the timezone with the date command (display the current timezone)

### Firewall

1. Block all incoming connections on all ports: ``` ufw default deny incoming ```
2. Allow outgoing connection on all ports: ``` ufw default allow outgoing ```
3. Allow incoming connection for SSH on port 2200: ``` ufw allow 2200/tcp ```
4. Allow incoming connections for HTTP on port 80: ``` ufw allow www ```
5. Allow incoming connection for NTP on port 123: ``` ufw allow ntp ```
6. Check the rules that have been added: ``` ufw show added ```
7. Enable the firewall: ``` ufw enable ```
8. Check firewall status: ``` ufw status ```

### Note
> All steps above are made using ``` root ``` user
> Developer may also do the steps using **grader** User but you should start all command
> Using ``` sudo ```

### Connect Using ``` grader ``` User

1. Exit the current terminal to close the connection of the root user: ``` exit ```
2. Connect using ``` grader ``` user :
``` ssh -i ~/.ssh/udacity_key.rsa grader@{YOUR.IP.GOES.HERE} -p 2200 ```

### Update Installed Packages

1. Update the package indexes: ``` apt-get update ```

2. Upgrade the installed packages: ``` apt-get upgrade ```

3. Reboot the machine: ``` reboot ```

### Install Required Software

1. Apache to serve a Python mod_wsgi application

	* Install Apache: ``` sudo apt-get install apache2 ```
	* Install libapache2-mod-wsgi package: ``` sudo apt-get install libapache2-mod-wsgi ```
	* Enable mod_wsgi: ``` sudo a2enmod wsgi ```
	* Start apache service ``` sudo service apache2 start ```

2. Install, Configure PostgreSQL

	* ``` sudo apt-get install postgresql postgresql-contrib ```
	* Disable remote connections to PostgreSQL check: ``` sudo nano /etc/postgresql/9.3/main/pg_hba.conf ```
	*Only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6*
	* Create a PostgreSQL user called catalog: ``` sudo -u postgres createuser -P catalog ```
	* Create an empty database called catalog: ``` sudo -u postgres createdb -O catalog catalog ```

3. Other Requirements

	* Flask: ``` sudo apt-get install python-psycopg2 python-flask ```
	* SQLAlchemy: ``` sudo apt-get install python-sqlalchemy python-pip ```
	* Aauth2client: ``` sudo pip install oauth2client ```
	* Requests: ``` sudo pip install requests ```
	* Httplib2: ``` sudo pip install httplib2 ```
	* Flask-seasurf: ``` sudo pip install flask-seasurf ```
	* ```pip install werkzeug==0.8.3 ```
	* ```pip install flask==0.9 ```
	* ```pip install Flask-Login==0.1.3 ```
	* Git: ``` sudo apt-get install git ```
	* Configure your username: ``` git config --global user.name <username> ```
	* Configure your email: ``` git config --global user.email <email> ```

4. Clone and Configure Project

	* Navigate to host directory: ``` cd /var/www/ ```
	* Create catalog directory: ``` sudo mkdir catalog ```
	* create wsgi configuration file: ``` sudo touch catalog.wsgi ```
	* Initialize wizgi file:
	```python

	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0, '/var/www/catalog/')

	from catalog import app as application

	application.secret_key = 'YOUR_SECRET_KEY'

	```
	* clone the project: ``` sudo git clone https://github.com/elmasria/item-catalog.git```

5. Configure Apache

	* Create a virtual host conifg file: ``` sudo nano /etc/apache2/sites-available/catalog-app.conf ```

	```
	<VirtualHost *:80>
        ServerAdmin webmaster@localhost

        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /var/www/catalog/catalog/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /var/www/catalog/catalog/static

        # Allow Apache to serve the files from the static directory
        <Directory  /var/www/catalog/catalog/static/>
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>

	```
	* Disable the default virtual host with: ``` sudo a2dissite 000-default.conf ```
	* Enable the catalog app virtual host: ``` sudo a2ensite catalog-app.conf ```
	* Reload Apache: ``` sudo service apache reload ```

6. Tuning

	* Update the Google OAuth client secrets file
	* Update the Facebook OAuth client secrets file
	* Update DATABASE_URL ``` postgresql://catalog:PASSWORD@localhost/catalog ```
	* Rename app.py to __init__.py
	* Setup database: ``` sudo python setup.py ```
	* Initialize database: ``` sudo python iniDB.py ```
	* Visit Application:
	``` http://ec2-35-160-12-62.us-west-2.compute.amazonaws.com/ ```
	``` http://35.160.12.62 ```

## References

1. [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299)
2. [Ubuntu Upgrades](https://wiki.ubuntu.com/Security/Upgrades)
3. [ubuntu PostgreSQL](https://help.ubuntu.com/community/PostgreSQL)
4. [Resolve host error](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)
5. [Deploy a Flask Application](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
6. [Getting Started Guide](https://docs.google.com/document/d/1J0gpbuSlcFa2IQScrTIqI6o3dice-9T7v8EDNjJDfUI/pub?embedded=true)