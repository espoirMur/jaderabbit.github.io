---
layout: post
title: "install nvidia driver"
permalink: install-nvidia-driver
date: 2022-03-27 13:36:01
comments: true
description: "How I fix this issue NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running"
keywords: ""
categories:
layout: post
tags:

---

I had many driver installed I my virtual machine , so It was actually the reason why I was having the error.

To fix it I had first to remove all driver I have installed before using :
- `sudo apt-get purge nvidia-*`
- `sudo apt-get update`
-`sudo apt-get autoremove`

After that I when a head and installed the latest version of it nvidia driver:

I did :

- `apt search nvidia-driver` 
To get the latest version of the driver
After getting the latest version I installed it with :

Edit Sept 2021 : According to the last comment  by @a-r-j  you can install a couple of dependencies before 
   * `sudo apt install libnvidia-common-470`
   * `sudo apt install libnividia-gl-470`

Then you can move forward and install the driver. 

   * `sudo apt install nvidia-driver-470`


And after installing it I rebooted my machine and checked with :
  
 `nvidia-smi`
 
 And tata ☄️
 
 The results :
 
 https://imgur.com/a/xfpvrtb
 
 Ressources :
 
 https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/
 
  
  
  
