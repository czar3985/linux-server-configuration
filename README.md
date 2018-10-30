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

check the users that are allowed to use sudo:
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

## The hosted web application

_To follow..._


## References

1. [Udacity](https://www.udacity.com/)'s course "Configuring Linux Web Servers"
2. http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately