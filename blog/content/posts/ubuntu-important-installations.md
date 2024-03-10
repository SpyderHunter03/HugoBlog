---
title: "Ubuntu 20.04 LTS Installations"
date: 2024-03-09T21:53:00-04:00
image: "images/post/ubuntu20.04lts.png"
description: "Important Installations on Ubuntu"
categories: ["code", "ubuntu20.04", "proxmox"]
draft: false
---

In the past few weeks, I have had to replace my server in my home that I was using with Proxmox.  Since I have done that, I have created several Proxmox VMs running Ubuntu 20.04LTS and had to install many applications on there, some of them several times.  Here is a non-comprehensive list of programs that I have installed and how to do that:

# First Things First

Always make sure you are running the update command before running any install scripts.  That way you have the most up to date packages.  Also, the update command will tell you how many packages are ready for upgrading as well.

`sudo apt update`

# Docker

Docker is used on over half of my servers.  I am using it to run several different programs, including nginx and a UPS management system that I am running.  I am not going to cover all the docker installations that I have in this post, but I am simply covering how to actually install docker in Ubuntu 20.04LTS:

`sudo apt install docker.io -y`

This will install Docker from apt using the root user.  You may have to enter the root password after this command if you haven't entered it recently.  The `-y` at the end bypasses the question of "Are you sure you want to install this?" that is asked at the end of most `apt install` calls.

# MySql

MySql can be installed via either docker if you are going to use it in a docker-compose manner, or you can install it via a package on the actual system.  This is how you install it on the actual system:

`sudo apt install mysql-server -y`

This will install Docker from apt using the root user.  Next thing that should be run is the mysql secure installation script, however first we need to make sure it doesn't break our system when we go to install it.  So first we must log into MySql and change the way the root user authenticates with MySql:

`sudo mysql`

`mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

`mysql> exit`

Next we can run the secure installation script:

`sudo mysql_secure_installation`

You can run through all the parameters with Yes reponses.  Here are some things to make sure of however:

- VALIDATE PASSWORD COMPONENT makes sure that all users created have some level of password security.  I recommend level 1 (Medium) but you can do level 2 (Strong) for a more secure password or level 0 (Low) if you are already securing things else where in your system
- ROOT PASSWORD sets the password for the root user.  This will be the password going forward and should be selected carefully, especially if you plan on allowing remote root control (which I don't recommend).
- REMOTE ROOT CONTROL sets an allowance for logging in as root on the database from somewhere other than localhost.  This can be helpful on a homelab, but dangerous if that server gets authorized to be open to the internet.

Next we want to create a new user in the system that will do all our work for us.  So we can log into the system and first thing we are going to do is move the root user authentication back to what it should be:

`mysql -u root -p`

`mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;`

Next we can create the user:

`mysql> CREATE USER '{username}'@'{accesslocation}' IDENTIFIED BY '{password}';`

Here are the brief explanations of the variables above:

- {username}: The name you want to log in as
- {accesslocation}: How you want the user to log in.  Can be set to `localhost` if you want only access locally on the server, or you can set it to `%` to allow access from anywhere.
- {password}: This is the password you want to login with.  Make sure it conforms to the password security level you set above.

Next you want to allow that user to access certain databases and tables.  Here is how to do that:

`mysql> GRANT {privileges} on {database}.{table} TO '{username}'@'{accesslocation}' WITH GRANT OPTION;`

Here are the brief explanations of the variables above:

- {privileges}: The privileges to assign to the user.  Here are some examples: `CREATE, ALTER, DROP, INSERT, UPDATE, INDEX, DELETE, SELECT, REFERENCES, RELOAD`.
- {database}: Database to apply privileges to for the user.  Use `*` for all databases on server.
- {table}: Table to apply privileges to for the user.  Use `*` for all tables in database.
- {username}@{accesslocation}: Make sure this is the same as the user created above.

Lastly make sure you flush the permissions:

`mysql> FLUSH PRIVILEGES;`

`mysql> exit;`

## Allowing Remote Access To Server

Now that we have MySql setup on the server, and things internal to the server can access the server like applications and such, but what if we want to remotely monitor the server from another computer, we need to allow access into the computer.  So first we need to open the firewall:

`sudo ufw allow mysql`

Now that we have allowed the outside world to access the port for MySql but now we got to tell MySql to allow traffic from other IP addresses.  So first we need to go change the configuration file:

`sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

In that file, you will find a line that sets the bind-address to 127.0.0.1.  You need to change that line to 0.0.0.0:

`bind-address = 0.0.0.0`

After changing the MySql configuration, you need to restart the service:

`sudo systemctl restart mysql`

# Conclusion

I will fill in more installations as I remember them.  I will not cover everything I install on every Ubuntu 20.04LTS instance, only the things that I do several times over, as well as other important commands