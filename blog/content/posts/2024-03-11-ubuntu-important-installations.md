---
title: "Ubuntu 20.04 LTS Installations"
date: 2024-03-11T21:53:00-04:00
image: "images/post/ubuntu20.04lts.png"
description: "Important Installations on Ubuntu"
categories: ["code", "ubuntu20.04", "proxmox"]
draft: false
---

In the past few weeks, I have had to replace my server in my home that I was using with Proxmox.  Since I have done that, I have created several Proxmox VMs running Ubuntu 20.04LTS and had to install many applications on there, some of them several times.  Here is a non-comprehensive list of programs that I have installed and how to do that:

# First Things First

Always make sure you are running the update command before running any install scripts.  That way you have the most up to date packages.  Also, the update command will tell you how many packages are ready for upgrading as well.

`sudo apt update`

# CURL

CURL is the system by which many Linux servers get files from the internet or call certain endpoints.  To install CURL, use the following command:

`sudo apt install curl`

It's just that simple!

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

# GitHub Runner

GitHub Runners are the agents that are self-hosted to run GitHub Actions workflows.  Every runner is assigned a special key by GitHub and runs only the workflows for that specific repository.  You can access how to install these from the repository you want the agent to run for, by going to Settings > Actions > Runners > New self-hosted runner > Linux.  This may change over time, but this is the current instructions for installing a GitHub Actions runner:

`cd ~`

`mkdir actions-runner && cd actions-runner`

`curl -o actions-runner-linux-x64-2.314.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.314.1/actions-runner-linux-x64-2.314.1.tar.gz`

`tar xzf ./actions-runner-linux-x64-2.314.1.tar.gz`

That all downloads the runner package and puts the files in the actions-runner folder.  Next we have to configure the runner and then install it as a service.

`./config.sh --url {repository} --token {token}`

The {repository} and {token} will be given in the GitHub Runners site and are specific to the code workflows that it should run

## Run GitHub Runner as Service

You can run the GitHub Runner as a command line program, but the real benefit to it is running it as a service on the box.  Here is the script you must run to process it as a service:

`sudo ./svc.sh install`

Next you start the service and the rest is history.

`sudo ./svc.sh start`

If you ever need to uninstall the runner, you can delete it from the Runners page forcefully, or you can run the following commands from the server that the bot is on:

`sudo ./svc.sh stop`

`sudo ./svc.sh uninstall`

## Update GitHub Action Workflow

In the event that the workflow that you want to run on the newly installed runner has been running on a github-owned runner previously, you will need to change the `runs-on` option in the workflow to use the self-hosted runner:

`runs-on: self-hosted`

# DotNet

Running DotNet applications require the DotNet runtime to be installed on the system that you want to run the application on.  However, if you are wanting to build the project on that server, or you have a workflow runner (like the GitHub Runner above), then you also need to ensure that the SDK is installed on the server as well.  Here are the steps to install both of these items:

`cd ~/Downloads`

`wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb`

`sudo dpkg -i packages-microsoft-prod.deb`

`rm packages-microsoft-prod.deb`

First we must download the package signing key to the system.  That is done with the commands above.  Next we install the actual packages:

`sudo apt-get update && sudo apt-get install -y dotnet-sdk-8.0`

`sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-8.0`

The first command installs the SDK for building and the second command installs the runtime to actually run the programs you want.

## Run A DotNet Project as a Service

Running a DotNet application as a service is fairly simple, in that we just have to create a service file and put it with the other services to start it.  Here is an example of a service file I am using for a Discord Bot run on my server:

```ini
[Unit]
Description=DiscGolfBot Discord Bot

[Service]
# systemd will run this executable to start the service
# if /usr/bin/dotnet doesn't work, use `which dotnet` to find correct dotnet executable path
ExecStart=/usr/bin/dotnet /home/tristan/actions-runner/_work/DiscGolfBot/DiscGolfBot/publish/DiscgolfBot.dll
# to query logs using journalctl, set a logical name here
SyslogIdentifier=DiscGolfBot

# Use your username to keep things simple.
# If you pick a different user, make sure dotnet and all permissions are set correctly to run the app
# To update permissions, use 'chown yourusername -R /srv/HelloWorld' to take ownership of the folder and files,
#       Use 'chmod +x /srv/HelloWorld/HelloWorld' to allow execution of the executable file
User=tristan

# ensure the service restarts after crashing
Restart=always
# amount of time to wait before restarting the service
RestartSec=5

# This environment variable is necessary when dotnet isn't loaded for the specified user.
# To figure out this value, run 'env | grep DOTNET_ROOT' when dotnet has been loaded into your shell.
Environment=DOTNET_ROOT=/usr/lib64/dotnet

WorkingDirectory=/home/tristan/actions-runner/_work/DiscGolfBot/DiscGolfBot/publish/

[Install]
WantedBy=multi-user.target
```

- The Description is how you want the project to be displayed on the system logs.
- The ExecStart is the command we want to run in order to actually start the application.  It has dotnet run the `.dll`, and you must point dotnet to the location of that `.dll`.
- The SyslogIdentifier is how the logs and service would be referred to.
- The User is the user account that you want the service to run as.
- The Restart is how the service should handle restarts if something were to fail.
- The RestartSec is how often the service should try to restart.
- The Environment allows you to set certain environment variables for the program being run.
- The WorkingDirectory is the place where the dotnet action can find all of the files it needs to run the application

This file needs to be created with a name similar to `discgolfbot.service` (which is the name of the above file), and placed in the `/etc/systemd/system/` folder.  The first time you put the file in there, you need to make sure you run the following command to update the system controller

`sudo systemctl daemon-reload`

# Conclusion

I will fill in more installations as I remember them.  I will not cover everything I install on every Ubuntu 20.04LTS instance, only the things that I do several times over, as well as other important commands