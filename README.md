# Linux Server Configuration

The project takes a baseline installation of a Linux server
and prepares it to host web applications. A database server is 
installed and configured, security measures againsts a number of 
attack vectors are implemented and a previously coded web application
is deployed in it to make it publicly accessible.

This README walks the user through the steps of configuring and accessing the 
Linux web server using Amazon Lightsail. 

## Server Details
**Public IP Address:** 54.252.131.90

**SSH Port:** 2200

**URLs to the hosted web application:** 
- http://ec2-54-252-131-90.ap-southeast-2.compute.amazonaws.com/
- http://54.252.131.90
- http://54.252.131.90.xip.io

## Overview of the Steps
1. Set-up the server
- Start a new Ubuntu server instance on Amazon Lightsail.
- Set-up SSH access to the server.
2. Secure the server
- Update installed packages.
- Change the SSH port from 22 to 2200. 
- Configure the Lightsail firewall to allow access to port 2200.
- Configure the Uncomplicated Firewall (UFW) to only allow incoming 
connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
3. Give grader access
- Create a new user account named grader.
- Give grader the permission to sudo.
- Create an SSH key pair for grader using the ssh-keygen tool.
4. Prepare to deploy the project
- Configure the local timezone to UTC.
- Install and configure Apache to serve a Python mod_wsgi application.
- Install and configure PostgreSQL.
- Install git
5. Deploy the Item Catalog project
- Clone and set-up the Item Catalog project
- Set it up in your server so that it functions correctly when visiting 
your server’s IP address in a browser.

## Detailed Configuration Steps

### 1. Set-up the server
#### - Start a new Ubuntu server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/).
In [Amazon Lightsail](https://lightsail.aws.amazon.com/), log-in using your 
Amazon Web Service (AWS) account or create a new account. Follow the steps in creating a 
Lightsail instance. For the instance image, select **Linux/Unix platform**. For Blueprint, 
select **OS only** and **Ubuntu 16.x**. Select the First Month Free instance plan. 
Indicate a unique instance name. 
The public IP address will be displayed when the instance is created.

#### - Set-up SSH access to the server.

Click on the created instance. Scroll down to the **Account Page link** and click it.
Download the **SSH key**. 
From a terminal in your local machine, copy the downloaded file to `~/.ssh`:
```
$ mv ~/Downloads/LightsailDefaultPrivateKey-ap-southeast-2.pem ~/.ssh/lightsailDefault.pem
```
**SSH** into the Lightsail instance:
```
$ ssh ubuntu@54.252.131.90 -p 22 -i ~/.ssh/lightsailDefault.pem
```

### 2. Secure the server
#### - Update installed packages.
After logging into the Lightsail instance, update available package lists and
upgrade the installed packages:
```
ubuntu@ip-172-26-10-47:~$ sudo apt-get update
ubuntu@ip-172-26-10-47:~$ sudo apt-get upgrade
```
Press Enter when asked if you want to keep the version currently installed
for some packages.

#### - Configure the Lightsail firewall to allow access to port 2200.
In the web console of the Lightsail instance, go to **Networking - Firewall**. 
Add two more ports that accepts connections: 
```
Application: Custom, Protocol: TCP, Port range: 2200
Application: Custom, Protocol: UDP, Port range: 123
```

#### - Change the SSH port from 22 to 2200. Disable remote login for root.
In the terminal, while logged in to the Lightsail instance:
```
 ubuntu@ip-172-26-10-47::~$ sudo nano /etc/ssh/sshd_config
```
Locate the line that says: `Port 22` and replace `22` with `2200`. 
Locate the line that says: `PermitRootLogin` and replace `prohibit-password` with `no`
Save and exit by pressing Ctrl-O, Enter and Ctrl-X.

Restart the sshd service: 
```
 ubuntu@ip-172-26-10-47::~$ sudo service ssh restart
```

#### - Configure the Lightsail Uncomplicated Firewall (UFW)
Only allow incoming connections for SSH (port 2200), HTTP (port 80), 
and NTP (port 123).

Block all incoming connections and allow outgoing connections:
```
 ubuntu@ip-172-26-10-47::~$ sudo ufw default deny incoming
 ubuntu@ip-172-26-10-47::~$ sudo ufw default allow outgoing
```
Allow incoming connections for SSH (port 2200), HTTP (port 80) 
and NTP (port 123):
```
 ubuntu@ip-172-26-10-47::~$ sudo ufw allow 2200/tcp
 ubuntu@ip-172-26-10-47::~$ sudo ufw allow 80/tcp
 ubuntu@ip-172-26-10-47::~$ sudo ufw allow 123/udp
```
Enable the firewall.
```
 ubuntu@ip-172-26-10-47::~$ sudo ufw enable
```
Check all rules and that the firewall is active:
```
 ubuntu@ip-172-26-10-47::~$ sudo ufw status
```
Expected Result:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

### 3. Give grader access
#### - Create a new user account named grader.
```
ubuntu@ip-172-26-10-47:~$ sudo adduser grader
```

#### - Give grader the permission to sudo.
```
ubuntu@ip-172-26-10-47:~$ sudo nano /etc/sudoers.d/grader
```
In the file, add the following lines:
```
# User rules for grader
grader ALL=(ALL) NOPASSWD:ALL
```

#### - Create an SSH key pair for grader using the ssh-keygen tool.
In local terminal:
```
$ ssh-keygen
```
Enter filename where key will be saved. Ex: `/c/Users/pixie/.ssh/itemCatalog`

Open itemCatalog.pub and copy the public key:
```
$ cat ~/.ssh/itemCatalog.pub
```
In Lightsail terminal, change to user `grader`. Enter password indicated when user was added:
```
ubuntu@ip-172-26-10-47:~$ su grader
```
Go to the grader user's home. Create a `.ssh` directory. 
Create an `authorized_keys` file inside the `.ssh` directory. 
Open the file and paste the copied public key from `itemCatalog.pub`. 
Ctrl-O, Enter and Ctrl-X to save
```
grader@ip-172-26-10-47:/home/ubuntu$ cd ~
grader@ip-172-26-10-47:~$ mkdir .ssh
grader@ip-172-26-10-47:~$ touch .ssh/authorized_keys
grader@ip-172-26-10-47:~$ nano .ssh/authorized_keys
```
Change the permissions of the created directory and file:
```
grader@ip-172-26-10-47:~$ chmod 700 .ssh
grader@ip-172-26-10-47:~$ chmod 644 .ssh/authorized_keys
```
The grader may now log-in using ssh key. Enter passphrase.
```
$ ssh -p 2200 grader@54.252.131.90 -i ~/.ssh/itemCatalog
```

### 4. Prepare to deploy the project
#### - Configure the local timezone to UTC.
```
grader@ip-172-26-10-47:~$ sudo dpkg-reconfigure tzdata
```
Select 'None of these' and 'UTC' from the options.

#### - Install and configure Apache to serve a Python mod_wsgi application.
Install apache. In the browser, visit http://54.252.131.90. It should load Apache's default 
page. 
```
grader@ip-172-26-10-47:~$ sudo apt-get install apache2
```
Install the Python 3 version of the Apache WSGI module. Restart Apche to use the new module:
```
grader@ip-172-26-10-47:~$ sudo apt-get install libapache2-mod-wsgi-py3
grader@ip-172-26-10-47:~$ sudo service apache2 restart
```

#### - Install and configure PostgreSQL.
Install Postgres package and the additional utilities. 
```
grader@ip-172-26-10-47:~$ sudo apt-get install postgresql postgresql-contrib
```
Do not allow remote connections. Check the host based authentication file:
```
grader@ip-172-26-10-47:~$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
Confirm that the remote declarations (TYPE: host) apply to interfaces that specify the local machine. 
This is the current default when installing PostgreSQL from the Ubuntu repositories.
```
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Switch to the postgres user. Connect to the Postgres system:
```
grader@ip-172-26-10-47:~$ sudo su - postgres
postgres@ip-172-26-10-47:~$ psql
```
Create a database and user named `catalog` that has limited permissions to your catalog application database:
```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'udacity';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
postgres=# \q
postgres@ip-172-26-10-47:~$ exit
```

#### - Install git
```
grader@ip-172-26-10-47:~$ sudo apt-get install git
```

### 5. Deploy the Item Catalog project
The application can be accessed from the URL found with the nslookup command.
The URL and hostname is: `ec2-54-252-131-90.ap-southeast-2.compute.amazonaws.com`
```
grader@ip-172-26-10-47:~$ nslookup 54.252.131.90
```

#### - Create client secret for Google log-in
Follow the steps below to create _client_secrets.json_

1. In https://console.developers.google.com/apis/dashboard, 
sign in to your Google account
2. Create Project. Indicate a name for the app
3. Go to your app's page in Google APIs Console
4. Choose Credentials
5. Create an OAuth Client ID.
6. Configure the consent screen, with email and app name
7. Add authorized domains: 
- amazonaws.com
- xip.io
8. Follow the steps to verify ownership of the domain.
9. Choose Web application list of application types
10. Set the authorized JavaScript origins: 
- http://ec2-54-252-131-90.ap-southeast-2.compute.amazonaws.com
- http://54.252.131.90.xip.io
11. Authorized redirect URIs: 
- http://ec2-54-252-131-90.ap-southeast-2.compute.amazonaws.com/login
- http://ec2-54-252-131-90.ap-southeast-2.compute.amazonaws.com/gconnect
- http://54.252.131.90.xip.io/login
- http://54.252.131.90.xip.io/gconnect
12. Download the client secret JSON file

#### - The necessary modifications to the Item Catalog project
Prepare the item catalog project: 
a. Change code from Python 2 to Python 3. 
b. Change database from SQLite to PostgreSQL. 
c. Check that it is working well in Udacity's vagrant setup.
d. Rename the downloaded JSON file to `client_secrets.json` and place
in the same folder as the `pokemon_types.py` file
e. In code, change these lines: 
`from database_setup import XXX` and `from view_model import XXX` to
`from .database_setup import XXX` and `from .view_model import XXX`
f. In `client_secret.json` path in the code, prepend the current directory's path 
`os.path.dirname(__file__)`

#### - Set the code up in your server
Create the directory where the code will go. Change permissions. Install a virtual environment to work on.
Install the packages while inside the virtual environment. Set-up and populate the database by running
`database_setup.py` and `initial_entries.py`. Check that there are no errors when running the main
script `pokemon_types.py`: 
```
grader@ip-172-26-10-47:/var/www$ sudo mkdir ItemCatalog
grader@ip-172-26-10-47:/var/www$ cd ItemCatalog/
grader@ip-172-26-10-47:/var/www/ItemCatalog$ sudo git clone https://github.com/czar3985/pokemon-types-v2.git PokemonApp
grader@ip-172-26-10-47:/var/www/ItemCatalog$ cd ..
grader@ip-172-26-10-47:/var/www$ sudo chown -R root:root ItemCatalog/
grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo apt-get install python3 python3-pip
grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install --upgrade pip
grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install virtualenv 
grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo virtualenv venv
grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ source venv/bin/activate 
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install Flask 
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install sqlalchemy
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install sqlalchemy_utils
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install oauth2client 
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install psycopg2
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install psycopg2-binary
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install httplib2
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo pip3 install requests
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo python3 database_setup.py
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo python3 initial_entries.py
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo python3 pokemon_types.py
```
Create the conf file below. Disable the default conf file and enable the newly-created one for the
app:
```
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo nano /etc/apache2/sites-available/ItemCatalog.conf
---
<VirtualHost *:80>
    ServerName 54.252.131.90
    ServerAdmin ubuntu@54.252.131.90
    WSGIScriptAlias / /var/www/ItemCatalog/catalog.wsgi
    <Directory /var/www/ItemCatalog/PokemonApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/ItemCatalog/PokemonApp/static
    <Directory /var/www/ItemCatalog/PokemonApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
---
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog$ ls /etc/apache2/sites-enabled/
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo a2dissite 000-default
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ sudo a2ensite ItemCatalog
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog/PokemonApp$ cd /var/www/ItemCatalog
```
Create the wsgi file that runs the application as follows. Restart the apache service. Check the log if any
errors are encountered:
```
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog$ sudo nano catalog.wsgi
---
#!/usr/bin/env python
import sys
import logging

logging.basicConfig(stream=sys.stderr)

sys.path.append("/var/www/ItemCatalog/")
sys.path.append("/usr/local/lib/python3.5/dist-packages")
sys.path.append("/usr/lib/python3/dist-packages")

from PokemonApp.pokemon_types import app as application

application.secret_key = 'super_secret_key'
---
(venv) grader@ip-172-26-10-47:/var/www$ sudo chmod 755 -R ItemCatalog
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog$ sudo service apache2 restart 
(venv) grader@ip-172-26-10-47:/var/www/ItemCatalog$ sudo tail -100 /var/log/apache2/error.log
```

## Summary of Software Installed
`apache2`, `libapache2-mod-wsgi-py3`, `postgresql`, `postgresql-contrib`, `git`, 
`python3`, `python3-pip`, `virtualenv`, `Flask`, `sqlalchemy`, `sqlalchemy_utils`, 
`oauth2client`, `psycopg2`, `psycopg2-binary`, `httplib2`, `requests`

## Resources
1. [Udacity](https://www.udacity.com/)'s course "Configuring Linux Web Servers"
2. [Changing the SSH Port for Your Linux Server](https://nz.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306)
3. [How To Set Up a Firewall with UFW on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04)
4. [Write Python 3 Web Apps with Apache2 mod_wsgi](http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately)
5. [Creating user, database and adding access on PostgreSQL](https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e)
6. [How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
7. [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
8. [How To Install Git on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-16-04#how-to-set-up-git)
9. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
10. [How To Find Hostname From IP Address](https://javarevisited.blogspot.com/2011/09/find-hostname-from-ip-address-to.html)
11. [os.path — Common pathname manipulations](https://docs.python.org/3/library/os.path.html)
12. [PostgreSQL DROP DATABASE](http://www.postgresqltutorial.com/postgresql-drop-database/)
13. [Permission denied to generate login hint for target domain when hosted on AWS](https://stackoverflow.com/a/52294864)
14. [How to Disable Remote Logon for Root on Ubuntu 16.04 LTS Servers](https://websiteforstudents.com/how-to-disable-remote-logon-for-root-on-ubuntu-16-04-lts-servers/)