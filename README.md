# Linux Server Configuration

The project takes a baseline installation of a Linux server
and prepares it to host web applications. A database server is 
installed and configured, security measures againsts a number of 
attack vectors are implemented and a previously coded web application
is deployed in it to make it publicly accessible.

This README walks the user through the steps of configuring and accessing the 
Linux web server.


## Configuration Steps

### Install Linux
#### Optional Prerequisites
It is recommended but not required to set-up the server using a virtual environment.
1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds_5_2) 
version 5.2
2. Download and install [Vagrant](https://www.vagrantup.com/)

vagrant init, up, ssh

_Ubuntu installation_
the Ubuntu distribution is trusty64

### Required packages for the web and database server

_Apache and Postgresql installation_

### Preparing the code

1. Clone your project into the vagrant environment: 

Cloned project: https://github.com/czar3985/restaurant-flask

2. Delete the created database
3. Make the necessary mods to use python 3 instead of 2

ex. show code changes in restaurantmenu.py

3. Test that it runs and install the required packages (ex. SQLAlchemy, pip upgrade, python)

```
sudo -H apt-get install python3 python3-pip
sudo -H pip3 install --upgrade pip
sudo -H pip3 install flask 
sudo -H pip3 install sqlalchemy
sudo -H pip3 install oauth2client 
```
issues:
```
Found existing installation: six 1.5.2
Cannot uninstall 'six'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
```
to fix:
```
sudo -H pip3 install --ignore-installed six
```
and then:
```
sudo -H pip3 install oauth2client 
```
4. Switch from sqlite to PostgreSQL

Just create a user and database. Change the SQLAlchemy engine URL and SQLAlchemy does the rest.

Install postgre if not yet installed:
vagrant@vagrant:~$  sudo apt-get update
sudo apt-get install postgresql postgresql-contrib

Change to the postgre user:
sudo su - postgres

Enter postgresql app:
postgres@vagrant:/$ psql


postgres=# CREATE DATABASE restaurantMenu;
postgres=# CREATE USER admin;

Giving the user a password

psql=# ALTER ROLE admin WITH ENCRYPTED PASSWORD 'yourpass';

Granting privileges on database

psql=# GRANT ALL PRIVILEGES ON DATABASE restaurantMenu TO admin;
postgres=# \q
postgres@vagrant:/$ exit

In code, find all create_engine instances. change to postgresql database access:
change from:
engine = create_engine('sqlite:///restaurantmenu.db')
to 
engine = create_engine('postgresql://admin:<password>@localhost/restaurantmenu')
replace password portion

References:
https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e

## Server Access

Setup grader user:

logged in as vagrant:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo adduser grader
enter unix password: udacity
Full Name: Udacity Grader
Other info: blank
```

```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo adduser grader
Adding user `grader' ...
Adding new group `grader' (1003) ...
Adding new user `grader' (1003) with group `grader' ...
Creating home directory `/home/grader' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
        Full Name []: Udacity Grader
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```


to confirm:
finger grader
```
vagrant@vagrant-ubuntu-trusty-64:~$ finger grader
Login: grader                           Name: Udacity Grader
Directory: /home/grader                 Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

check that you can connect to the server as the new user from 
a local machine:
```
$ ssh grader@127.0.0.1 -p 2222
```
You may be asked to verify and for the grader's password. Enter
the password set earlier to log in

Note that if password authentication has been turned off, logging in 
using password will not be successful. 

If so, modify the password authentication part in the configuration file for
SSH connections. In vagrant machine:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo nano /etc/ssh/sshd_config
```
Set PasswordAuthentication to yes
```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication yes
```
Ctrl O, Enter, Ctrl X to save and exit
then restart the service:

```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 2011
```

Back to the vagrant machine, check the users that are allowed to use sudo:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo ls /etc/sudoers.d
vagrant should be one of the users.
ex output:
vagrant@vagrant-ubuntu-trusty-64:~$ sudo ls /etc/sudoers.d
90-cloud-init-users  README  student  vagrant
```

give the grader access to sudo:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader
```

Confirm addition of grader to the directory:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo ls /etc/sudoers.d         90-cloud-init-users  grader  README  student  vagrant
```
edit the grader file that was added:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo nano /etc/sudoers.d/grader
```
file contents:
```
# CLOUD_IMG: This file was created/modified by the Cloud Image bui$
vagrant ALL=(ALL) NOPASSWD:ALL
```
Ctrl-O, Enter, Ctrl-X to save and exit file

confirm that the file was modified:
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo cat /etc/sudoers.d/grader
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
grader ALL=(ALL) NOPASSWD:ALL
```
and
```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo cat /etc/passwd
```
should show at the end:
```
grader:x:1003:1003:Udacity Grader,,,:/home/grader:/bin/bash
```

Implement login via SSH key instead of password:

In local machine, generate key-pair:
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/pixie/.ssh/id_rsa): /c/Users/pixie/.ssh/serverConfig
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/pixie/.ssh/serverConfig.
Your public key has been saved in /c/Users/pixie/.ssh/serverConfig.pub.
The key fingerprint is:
XXXXXX
The key's randomart image is:
XXXXXX
```

Copy the public key from:
```
$ cat .ssh/serverConfig.pub
```

In the server, while logged in as grader, in home directory, create a .ssh folder and 
and authorized_keys file inside the .ssh folder:
```
$ ssh grader@127.0.0.1 -p 2222
grader@vagrant-ubuntu-trusty-64:~$ cd ~
grader@vagrant-ubuntu-trusty-64:~$ mkdir .ssh
grader@vagrant-ubuntu-trusty-64:~$ touch .ssh/authorized_keys
```

Edit the authorized_keys file and place public key there:
```
grader@vagrant-ubuntu-trusty-64:~$ nano .ssh/authorized_keys
```

Set the permissions:
```
grader@vagrant-ubuntu-trusty-64:~$ chmod 700 .ssh
grader@vagrant-ubuntu-trusty-64:~$ chmod 644 .ssh/authorized_keys
```

From a local terminal try logging in using the key-pair. It should be able to
log-in without using the grader account password
```
$ ssh grader@127.0.0.1 –p 2222 –i ~/.ssh/serverConfig
```
Enter passphrase set when generating the key pair


## The hosted web application

_To follow..._


## References

1. [Udacity](https://www.udacity.com/)'s course "Configuring Linux Web Servers"
2. http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately