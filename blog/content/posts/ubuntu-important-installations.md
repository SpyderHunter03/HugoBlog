---
title: "Ubuntu 20.04 LTS Installations"
date: 2024-03-09T21:53:00-04:00
image: "images/post/ubuntu20.04lts.png"
description: "Important Installations on Ubuntu"
categories: ["code", "ubuntu20.04", "proxmox"]
draft: false
---

In the past few weeks, I have had to replace my server in my home that I was using with Proxmox.  Since I have done that, I have created several Proxmox VMs running Ubuntu 20.04LTS and had to install many applications on there, some of them several times.  Here is a non-comprehensive list of programs that I have installed and how to do that:

# Docker

Docker is used on over half of my servers.  I am using it to run several different programs, including nginx and a UPS management system that I am running.  I am not going to cover all the docker installations that I have in this post, but I am simply covering how to actually install docker in Ubuntu 20.04LTS:

`sudo apt install docker.io -y`

This will install Docker from apt using the root user.  You may have to enter the root password after this command if you haven't entered it recently.  The `-y` at the end bypasses the question of "Are you sure you want to install this?" that is asked at the end of most `apt install` calls.

# Conclusion

I will fill in more installations as I remember them.  I will not cover everything I install on every Ubuntu 20.04LTS instance, only the things that I do several times over, as well as other important commands