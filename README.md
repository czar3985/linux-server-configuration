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

**URL to the hosted web application:**

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

Click on the created instance. Scroll down to the Account Page link and click it.
Download the SSH key. 
From a terminal in your local machine, copy the downloaded file to `~/.ssh`:
```
$ mv ~/Downloads/LightsailDefaultPrivateKey-ap-southeast-2.pem ~/.ssh/lightsailDefault.pem
```
SSH into the Lightsail instance:
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

In the web console of the Lightsail instance, go to Networking - Firewall. 
Add two more ports that accepts connections: 
```
Application: Custom, Protocol: TCP, Port range: 2200
Application: Custom, Protocol: UDP, Port range: 123
```

#### - Change the SSH port from 22 to 2200.

In the terminal, while logged in to the Lightsail instance:
```
 ubuntu@ip-172-26-10-47::~$ sudo nano /etc/ssh/sshd_config
```
Locate the line that says: `Port 22` and replace `22` with `2200`. 
Save and exit by pressing Ctrl-O, Enter and Ctrl-X.

Restart the sshd service: 
```
 ubuntu@ip-172-26-10-47::~$ sudo service ssh restart
```
#### - Configure the Lightsail Uncomplicated Firewall (UFW)
Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

Block all incoming connections and allow outgoing connections:
```
 ubuntu@ip-172-26-10-47::~$ sudo ufw default deny incoming
 ubuntu@ip-172-26-10-47::~$ sudo ufw default allow outgoing
```
Allow incoming connections for SSH (port 2200), HTTP (port 80) and NTP (port 123):
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

#### - Install and configure Apache to serve a Python mod_wsgi application.

#### - Install and configure PostgreSQL.

#### - Install git

### 5. Deploy the Item Catalog project
#### - Clone and set-up the Item Catalog project

#### - Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser


## Summary of Software Installed


## Resources

1. [Udacity](https://www.udacity.com/)'s course "Configuring Linux Web Servers"
2. http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately
3. https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e
4. https://nz.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306
5. https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04