---
layout: post
title: "Home Assistant Installation (VMWare Workstation)"
date: 2023-03-05 04:00:00 -0400
categories: home-lab
pin: true
tags: jekyll cd-tech ctn digital-equity tech-for-good
image: 
  path: /assets/img/posts/ha.jpg
  #lqip: data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD/
---
## Home Assistant Installation (VMWare Workstation)

In this setup of Home Assistant we will be using VMWare Workstation Pro.  To being we will start by setting up our virtual machine on the software.

Pre-requirements: An active license for VMWare Workstation Pro, 2 CPUs cores, 2GB of RAM, and 16GB of storage

Step 1) Open up VMWare Workstation Pro

Step 2) Go to File > New Virtual Machine… or by pressing Ctrl+N

Step 3) Use the default Typical (recommended) and click Next

Step 4) Select the option I will install the operating system later and click Next

Step 5) Select Linux and the version Other Linux 5.x and later kernel 64-bit then click Next

Step 6) Give the virtual machine a name of your pleasure then click Next

Step 7) Set the disk size to 16GB and click Next

Step 8) Click on Customize Hardware

Step 9) Increase the RAM to 2048MB or 2GB then hit Close

Step 10) Click Finish

Step 11) Go to Home Assistants webpage here and download the Vmware Workstation (.vmdk) file.

Step 12) Rename the downloaded file to the name home-assistant.vmdk.  We will then replace the home-assistant.vmdk file in the Virtual Machines folder with this newly downloaded one.

Step 13) Open the home-assistant.vmx file and add the following line under the .encoding line firmware="efi"

Step 14) You can now power on your virtual machine home-assistant

**Note:** If you get this popup box then you can click Yes to contine

Step 15) Once you get the Command Line Interface (CLI) for the home assistant below you are ready to navigate to http://IP_Address:8123 or http://homeassistant.local:8123.


You will be prompted with this screen to set up credentials if everything was successfully installed.