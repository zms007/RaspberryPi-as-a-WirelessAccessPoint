In this project, your Raspberry Pi will be connected to the internet via RJ45 network cable. 
Turn your Raspberry Pi into a wireless access point, you can make it act as a router. 
Use Raspbian and install several packages that gives the Pi the ability to do router-like things such as assign IP addresses to devices that connect to it. 
Configure the your tablet so that it can surf the internet through your Raspberry Pi wireless access point. 

The Raspberry Pi is a versatile device, and with the latest models now equipped with built-in wireless features, its capabilities have expanded even further. 
One interesting use is turning the Raspberry Pi into a wireless access point, allowing it to function like a router. 
Although it may not be as powerful as commercial routers, it works well for small projects or personal use. 
Setting up a Raspberry Pi as an access point is also an enjoyable and educational experience, giving you the chance to explore networking in a practical way.

How to Set Up Your Raspberry Pi as a Wireless Access Point?
-----
In this project, we will be using some command-line instructions, but don’t worry — it’s quite simple to follow. 
Basically, we are going to use Raspberry Pi OS (formerly known as Raspbian) and install a few software packages. 
These packages will enable the Raspberry Pi to act like a router by allowing it to assign IP addresses to connected devices and manage network traffic.

Step 1: Install and Update Raspberry Pi OS
-----
First, make sure you have Raspberry Pi OS installed on your Raspberry Pi. You can refer to a full guide on how to install Raspberry Pi OS if you are not sure how to do this.

Once everything is set up and connected, open the terminal and run the following commands to update and upgrade your system:

sudo apt-get update  
sudo apt-get upgrade  

If there are any upgrades installed, it is recommended to reboot your Raspberry Pi by typing:

sudo reboot  

This will ensure all updates are properly applied before moving to the next step.

Step 2: Install hostapd and dnsmasq
-----
To turn your Raspberry Pi into a wireless access point, you need to install two important packages: hostapd and dnsmasq. Open the terminal and enter the following commands:

sudo apt-get install hostapd  
sudo apt-get install dnsmasq  

When prompted, press y to confirm the installation.

hostapd allows the Raspberry Pi to create a wireless hotspot, while dnsmasq provides simple DHCP and DNS services to manage IP addresses and connections.

Before we start configuring these programs, let’s stop them from running to avoid any issues:

sudo systemctl stop hostapd  
sudo systemctl stop dnsmasq  

Now they are ready for configuration in the next step.

Step 3: Set a Static IP Address for wlan0 Interface
-----
In this step, we will assign a static IP address to the wlan0 interface, which is the wireless adapter on the Raspberry Pi. For this example, we will use a typical home network range like 192.168.x.x, and set 192.168.0.10 for our access point.

To do this, we need to edit the dhcpcd configuration file. Open the file using this command:

sudo nano /etc/dhcpcd.conf  

Once inside the file, scroll to the bottom and add these lines:

interface wlan0  
static ip_address=192.168.0.10/24  
denyinterfaces eth0  
denyinterfaces wlan0  

Note: The last two lines are important for when we set up a network bridge later (explained in Step 8).

After adding the lines, press Ctrl + X to exit, then Y to confirm saving, and Enter to return to the terminal.

Your wlan0 now has a fixed IP address ready for the access point setup.

Step 4: Set Up the DHCP Server (dnsmasq)
-----
Now, we will configure dnsmasq to act as a DHCP server. A DHCP server is used to automatically assign IP addresses and other network settings to devices that connect to the access point.

Since the default dnsmasq configuration file has many unnecessary settings, it’s easier to create a new one from scratch. First, let’s rename the original file and then create a new configuration file:

sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig  
sudo nano /etc/dnsmasq.conf  

By doing this, we keep the original file safe and work with a clean file that dnsmasq will now use.

Next, type the following lines in the new file:

interface=wlan0  
dhcp-range=192.168.0.11,192.168.0.30,255.255.255.0,24h  

These settings tell dnsmasq to assign IP addresses between 192.168.0.11 and 192.168.0.30 to devices connecting through wlan0, with each lease lasting 24 hours.

After adding these lines, press Ctrl + X to exit, then Y to save, and Enter to confirm.

Your DHCP server is now ready to hand out IP addresses to connected devices.








