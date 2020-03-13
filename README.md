# raspberry-pi-captive-portal-guide
How to make a Raspberry Pi Captive Portal hosting a PHP server and everything.

### Description
With this device/configuartion, you can use it to trick users in giving their own credentials by hosting a copy of whatever site it is you're trying to gain access too. In my case, I wanted to see if it was possible to create a clone of the Starbucks access point to collect possible email address just for the fun of it. Setting it up this way will prompt the user to 'sign in' once the device is connected to the pi. When they sign in it sends them to a site you can control in any way possible. The data sent from there can go practially anywhere you want it to.

## Parts
* Raspberry Pi (Preferably 3)
* Alfa Wireless Card
* SD Card (Can be at least 4GB) / Host computer to install Raspbian
* Nessary Cables and Case if needed

# Installation Guide
(Use the [Raspbian Buster Lite Version](https://www.raspberrypi.org/downloads/raspbian/))
Also, I'm assuming you know how to get SSH access to it once its boot. Maybe I'll add that in here later.

## Installing Software Needed

``sudo apt-get update -y&& sudo apt-get upgrade -y``

``sudo apt-get install hostapd git dnsmasq libmicrohttpd-dev isc‐dhcp‐server iptables -y``

## Configuring DHCP
``sudo nano /etc/sysctl.conf``
Uncomment the following:
``net.ipv4.ip_forward=1``
``net.ipv6.conf.all.forwarding=1``

``sudo iptables ‐t nat ‐A POSTROUTING ‐o wlan0 ‐j MASQUERADE``
This is wlan0 because we are forwarding things to the internet this way. If you wanted enternet to be used then do eth0.

``sudo apt‐get install iptables‐persistent``


## Configuring hostapd

``sudo nano /etc/hostapd/hostapd.conf``
```
interface=wlan1
ssid=Not Starbucks WiFi
hw_mode=g
channel=11
macaddr_acl=0
```
Put that (the above text) in the hostapd.conf file. If you are doing ethernet forwarding, use the wifi interface on the pi (wlan0). Depending on the type of wifi card you're using these settings could vary.

``sudo nano /etc/default/hostapd``
``DAEMON_CONF="/etc/hostapd/hostapd.conf"``
Same thing.

``sudo systemctl start hostapd``
Start hostapd
Once started you should be able to see the access point from any WiFi enabled device.

## Installing and Configuring [NoDogSplash](https://github.com/nodogsplash/nodogsplash)

``git clone https://github.com/nodogsplash/nodogsplash.git``

``cd ~/nodogsplash``

``make``

``sudo make install``

``sudo nano /etc/nodogsplash/nodogsplash.conf``
Put this inplace of what is in the configuartion.
```
GatewayInterface wlan1
GatewayAddress 192.168.4.1
MaxClients 250
use_outdated_mhd 1 *
```
For this, I used another wireless card since the device is being a MITM captive portal. Make sure its the interface that supports monitor/AP mode. If its seperate from the PI it will be wlan1 in most cases. The IP should be the IP of the PI on the subnet that its hosting for users to connect. If you're forwarding ethernet then use wlan0.

``sudo nodogsplash``
Once this starts you'll be able to see devices connecting to your device. 
* If theres a bug and it breaks, add this line (Specified by *)


## Sources
* [PiMyLifeUp](https://pimylifeup.com/raspberry-pi-captive-portal/)
* [Ian from cybersecurityguy.com](http://cybersecurityguy.com/Building_a_Raspberry_Pi_Captive_Portal_Wi-Fi_Hotspot.pdf)
* [Trevor Phillips](https://trevphil.com/posts/captive-portal)
* [Raspberry Pi StackExchange](https://raspberrypi.stackexchange.com/questions/88438/raspberry-pi-as-access-point-with-captive-portal)
* [Documentation of NoDogSplash](https://nodogsplash.readthedocs.io)
* [Bug with Version in NoDogSplash](https://raspberrypi.stackexchange.com/questions/108803/issue-with-nodogsplash-saying-it-needed-updateed-libmicrohttpd-dev-but-i-seems)

