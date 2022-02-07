# Zero-Touch-Provisioning
Cisco ZTP on Catalyst 9300


ZTP with Mario G.
In this tutorial, I'm going to show you how to setup Cisco Zero Touch Provisioning (ZTP)
 
Introduction:  In my case, I'm using a ubuntu laptop as the DHCP server and also an HTTP server; I'm going to show you how to configure both and get the Zero Touch Provisioning up and running

First thing that you need to do is configure your DHCP server:

Step 1: Install DHCP Server
The first thing we need to do is install the dhcpd server by running the following command:
sudo apt install isc-dhcp-server -y

Step 2 Modify the DHCP Server:
Go to: sudo nano etc/dhcp/dhcpd.conf 

And you will be able to modify the dhcp server

# option definitions common to all supported networks...
option domain-name "cisco.com";
option domain-name-servers 192.168.1.2, 8.8.8.8;
INTERFACESv4="eno1";
default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# No service will be given on this subnet, but declaring it helps the
# DHCP server to understand the network topology.

subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.60 192.168.1.254;
      host sw-9300 {
         fixed-address                   192.168.1.65;
         hardware ethernet               ac:7a:56:b5:b1:80;
         option bootfile-name            "http://192.168.1.2/ztp-simple.py";
     }
  }





Step 3: Check the status of the DHCP server you can type the following command:

Systemctl status isc-dhcp-server

NOTE: if you don't have your ethernet cable connected from the laptop to the switch Mgtm interface your dhcp server it's going to fail see the screenshot below

NOTE: If you are having an error on your DHCP server do the following to find out what it's the issue:



Journalctl _PID=1   OR the number that the error that you are getting.



Step 4: After you have your DHCP server configure you will need to move to configure a HTTP server 
In my case I'm using Apache2 

sudo apt update
sudo apt install apache2

Note: After you create your apache you will need to load the Python script in the fallowing directory:
/var/www/html/ 

This is the location that the switch it's going to use to grab the python script.

To check the status of the Apache2 server you can type the following command:

Systemctl status apache2.service

Step 5: Create your python script and load it to the path above in the http server:


If you are in the directory you can type /var/www/html

You can type the following: nano ztp-simple.py and can copy and paste the below script:


#!/usr/bin/python3
print ("\n\n *** Sample ZTP Day0 Python Script *** \n\n")

# Importing cli module
import cli


print ("\n\n *** Executing show platform  *** \n\n")
cli_command = "show platform"
cli.executep(cli_command)

print ("\n\n *** Executing show version *** \n\n")
cli_command = "show version"
cli.executep(cli_command)

print ("\n\n *** Configuring a Loopback Interface *** \n\n")
cli.configurep(["interface loop 100", "ip address 10.10.10.10 255.255.255.255", "end"])


print ("\n\n *** Executing show ip interface brief  *** \n\n")
cli_command = "sh ip int brief"
cli.executep(cli_command)

print ("\n\n *** ZTP Day0 Python Script Execution Complete *** \n\n")

NOTE: make sure you change the permission to the file by typing:

mfg@makina:/var/www/html$ chmod 777 ztp-simple.py after you create the script 

mfg@makina:/var/www/html$ ls -l
total 20
-rwxrwxrwx 1 root root 10918 Jan 29 23:22 index.html
-rwxrwxrwx 1 root root   674 Jan 30 16:03 ztp-simple.py




Credits to:
From <https://ubuntu.com/tutorials/install-and-configure-apache#2-installing-apache> 

From <https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/175/b_175_programmability_cg/m_175_prog_ztp.html> 

How to configure DHCP on Ubuntu:
https://www.linuxfordevices.com/tutorials/ubuntu/dhcp-server-on-ubuntu

Apache2 directory: 
/var/www/html/ 

From <https://askubuntu.com/questions/683953/where-is-apache-web-root-directory-on-ubuntu> 

One of the explanation 
https://github.com/cisco-ie/IOSXE_ZTP

https://developer.cisco.com/docs/ios-xe/#!zero-touch-provisioning/dhcp-server-running-on-a-switch
![image](https://user-images.githubusercontent.com/20927485/152719264-d2f3b3a7-9f83-4cbb-ae2a-c89786a1ad6d.png)



Best YouTube video about DHCP on Ubuntu: How to Install DHCP Server in Ubuntu Server 20.04
![image](https://user-images.githubusercontent.com/20927485/152719216-2fd3c0e9-8ded-4b49-96f2-6bb68dc279da.png)
