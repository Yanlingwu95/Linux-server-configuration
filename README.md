# Linux Server Configuration

## Overview

This is the sixth project for the Udacity Full Stack Nanodegree. This project is about taking a baseline installation of a Linux server and prepare it to host my web applications.  We secured my server from a number of attack vectors, install and configure a database server, and deploy one of my existing web applications onto it. 

## Server Info

- **Public IP:** 52.4.173.75
- **Port:** 2200

## Getting Started

Use [Amazon Lightsail](https://amazonlightsail.com/) to create a Linux server instance. 

- Start a new Ubuntu Linux server instance on Amazon Lightsail.

  - Sign up for an account if you do not have an account, or log in. 
  - Create an instance.
  - Pick your instance image --> Linux/Unix --> OS Only --> Ubuntu(16.04 LTS).
  - Choose your instance plan (lowest tier is fine).
  - Give your instance a hostname.
  - Wait for startup, it takes several minutes. 
  - Once the instance has started up, follow the instruction provided to SSH into your serve.  If you are a windows user, click [here](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh) for the instruction. But from my own experience, the Ubuntu system is better to use. 

- Try to add one user in old port 22 first： 

  - Create a new user account named grader.

    `sudo adduser grader`

  - Give grader the permission to sudo.

    ```
         ssh-keygen -t rsa
          
      To login
          ssh -i .ssh/grader_udacity grader@52.4.173.75 -p 22
    ```

  - Give grader the permission to sudo.

    ```
      sudo vim /etc/sudoers.d/grader
    ```

    add text `grader ALL=(ALL) NOPASSWD:ALL`

- Secure your serve: 

  - Update all currently installed packages. 

    ```
     sudo apt-get update
      sudo apt-get upgrade
    ```

  - Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

    ```
      sudo vim /etc/ssh/sshd_config
    ```

    change port form 22 to 2200

  - Configure the Uncomplicated Firewall (UFW) to only allow incoming  connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

    ```
      sudo ufw allow 2200/tcp
      sudo ufw allow 80/tcp
      sudo ufw allow 123/tcp
      sudo ufw enable
    ```

    **Warning: ** When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app `Connect using SSH'`button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.