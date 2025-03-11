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

Step 5: Set Up the Access Point Software (hostapd)
-----
Now it’s time to configure hostapd, the software that will turn your Raspberry Pi into a wireless access point. First, let’s create and edit the hostapd configuration file. Open the file by typing:

sudo nano /etc/hostapd/hostapd.conf  

This will open a new file. In this file, type the following settings:

interface=wlan0  
bridge=br0  
hw_mode=g  
channel=7  
wmm_enabled=0  
macaddr_acl=0  
auth_algs=1  
ignore_broadcast_ssid=0  
wpa=2  
wpa_key_mgmt=WPA-PSK  
wpa_pairwise=TKIP  
rsn_pairwise=CCMP  
ssid=NETWORK  
wpa_passphrase=PASSWORD  

⚙️ Note: Replace NETWORK with your desired Wi-Fi name and PASSWORD with a strong password. These will be used to connect devices to your Raspberry Pi’s wireless network.

After entering the settings, press Ctrl + X to exit, then press Y to save, and Enter to confirm.

Link hostapd to its Configuration File
Next, we need to tell the system where to find this hostapd configuration file. To do this, open the hostapd default settings file:

sudo nano /etc/default/hostapd 

Look for this line:

#DAEMON_CONF=""

Remove the # at the beginning to uncomment the line, and add the path to the config file, so it looks like this:

DAEMON_CONF="/etc/hostapd/hostapd.conf"  

By removing the #, you activate this line and correctly link hostapd to the config file we just created.

Once done, press Ctrl + X, then Y, and Enter to save and exit.

Now hostapd knows where to find its settings when we start the service!

Step 6: Enable Traffic Forwarding
-----
In this step, we will set up traffic forwarding so that any device connected to your Raspberry Pi's Wi-Fi can access the internet through the Ethernet connection. This means that the wlan0 interface (Wi-Fi) will forward traffic through the Ethernet cable connected to your modem or router.

To do this, we need to edit a system configuration file. Open the sysctl.conf file using this command:

sudo nano /etc/sysctl.conf  

Inside the file, look for this line:

#net.ipv4.ip_forward=1  

Remove the # at the beginning of the line so that it becomes:

net.ipv4.ip_forward=1  

This change will enable IP forwarding, allowing the Raspberry Pi to pass network traffic between interfaces.

Once done, press Ctrl + X to exit, then Y to save, and Enter to confirm.

Now, your Raspberry Pi is ready to forward traffic from Wi-Fi to Ethernet.

Step 7: Add IP Masquerading with iptables
-----
Now, we will configure iptables to enable IP masquerading, which allows devices connected to your Raspberry Pi's Wi-Fi to share the Ethernet connection for internet access.

First, enter this command to add the masquerade rule for outbound traffic on eth0 (Ethernet):

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  

Next, save this iptables rule so that it remains active after reboot:

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"  

To make sure this rule loads automatically every time the Raspberry Pi starts, we need to edit the /etc/rc.local file. Open the file with:

sudo nano /etc/rc.local  

Before the line that says exit 0, add this line:

iptables-restore < /etc/iptables.ipv4.nat  

So, it will look something like this:

iptables-restore < /etc/iptables.ipv4.nat  
exit 0  

Finally, press Ctrl + X to exit, then Y to save, and Enter to confirm.

With this step completed, your Raspberry Pi will forward network traffic properly when it boots up.

Step 8: Enable Internet Connection Sharing
-----
At this stage, your Raspberry Pi can act as a wireless access point that other devices can connect to. However, these devices still can't access the internet through the Pi yet. To allow internet access, we need to create a network bridge that will pass all traffic between the wlan0 (Wi-Fi) and eth0 (Ethernet) interfaces.

To create this bridge, first install the required package:

sudo apt-get install bridge-utils  

Once installed, let’s create a new bridge called br0 by typing:

sudo brctl addbr br0  

After that, link the eth0 interface to the bridge:

sudo brctl addif br0 eth0  

Next, we need to configure the bridge settings. Open the /etc/network/interfaces file:

sudo nano /etc/network/interfaces  

At the end of the file, add these lines:

auto br0  
iface br0 inet manual  
bridge_ports eth0 wlan0  

These settings will connect both Ethernet and Wi-Fi interfaces to the same network bridge, allowing traffic to flow between them.

Once added, press Ctrl + X to exit, then Y to save, and Enter to confirm.

Now, when devices connect to the Raspberry Pi's Wi-Fi, they will be able to access the internet using the Pi’s Ethernet connection.

Step 9: Reboot the Raspberry Pi
-----
Once everything is set up, it’s time to restart your Raspberry Pi to apply all the configurations. You can reboot by typing this command:

sudo reboot 

After the reboot, your Raspberry Pi should now function as a wireless access point. You can test it by using another device, such as a laptop or smartphone, and searching for the Wi-Fi network name (SSID) that you created earlier in Step 5.

If everything is correct, you should be able to connect to the Raspberry Pi's Wi-Fi and access the internet!
