# Linux Server Configuration

The project takes a baseline installation of a Linux server
and prepares it to host web applications. A database server is 
installed and configured, security measures againsts a number of 
attack vectors are implemented and a previously coded web application
is deployed in it to make it publicly accessible.

This README walks the user through the steps of configuring and accessing the 
Linux web server using Amazon Lightsail. 

## Server Details
**IP Address:** 13.238.185.29

**SSH Port:**

**URL to the hosted web application:**

## Overview of the Steps
1. Set-up the server
- Start a new Ubuntu server instance on Amazon Lightsail.
- Set-up SSH access to the server.
2. Secure the server
- Update installed packages.
- Change the SSH port from 22 to 2200. 
- Configure the Lightsail firewall to allow it.
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

**- Start a new Ubuntu server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/).**

In [Amazon Lightsail](https://lightsail.aws.amazon.com/), log-in using your 
Amazon Web Service (AWS) account or create a new account. Follow the steps in creating a 
Lightsail instance. For the instance image, select **Linux/Unix platform**. For Blueprint, 
select **OS only** and **Ubuntu**. Select the **First Month free** instance plan. 
Indicate a unique instance name. 
The public IP address will be displayed when the instance is created.

**- Set-up SSH access to the server.**

### 2. Secure the server
**- Update installed packages.**

**- Change the SSH port from 22 to 2200.**

**- Configure the Lightsail firewall to allow it.**

**- Configure the Uncomplicated Firewall (UFW) to only allow incoming 
connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).**

### 3. Give grader access
**- Create a new user account named grader.**

**- Give grader the permission to sudo.**

**- Create an SSH key pair for grader using the ssh-keygen tool.**

### 4. Prepare to deploy the project
**- Configure the local timezone to UTC.**

**- Install and configure Apache to serve a Python mod_wsgi application.**

**- Install and configure PostgreSQL.**

**- Install git**

### 5. Deploy the Item Catalog project
**- Clone and set-up the Item Catalog project**

**- Set it up in your server so that it functions correctly when visiting 
your server’s IP address in a browser.**


## Summary of Software Installed


## Resources

1. [Udacity](https://www.udacity.com/)'s course "Configuring Linux Web Servers"
2. http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately
3. https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e