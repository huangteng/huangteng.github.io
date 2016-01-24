title: how to share wifi internet to Linux Embedded system board
date: 2015-06-20 10:43:51
tags:
toc: true
---


This post describes the configure procedure for Linux network settings, to achieve that the laptop can get access to the internet (through WLAN) while at the same time the development board (or any other device which has LAN port) can talk with it (ping, ftp, telnet, etc).

<br><br>

## Configure the LAN and WLAN (general description)
### Configure the LAN in terminal
``` bash
$ sudo nano /etc/network/interfaces
```
Configure the WLAN to dhcp and automatic and LAN with a static ip address

``` bash
auto lo
iface lo inet loopback

# The primary network interface
auto wlan0
iface wlan0 inet dhcp

# The second eth0
auto eth0
iface eth0 inet static
address 192.168.1.73

```
### restart the network interface
``` bash
$ sudo /etc/init.d/networking restart
```
Here if this cannot restart successfully, then try stop/start seperately, and the problem might be:
``` bash
stop: Unknown instance:
networking stop/waiting
```
and even in the network application in ubuntu, the Wireless configuration is also disabled, it means that at the moment, the /etc/network/interfaces cannot correctly configure the network, here the solution is to use network manager service to configure the network:
``` bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```
change the line managed=false to managed=true (normally NetworkManager does not manage interfaces that appear in /etc/network/interfaces), then restart this service:
``` bash
sudo service network-manager restart
```
Another method is you can keep managed=false, and just delete all the self-defined configurations in /etc/network/interfaces(keep the first 2 lines for the auto lo), and use the above command to restartd the network manager, because if network manager finds out that there is no configuration in the interfaces file, it will still handle the network configurations even it is set to false.

In Ubuntu, I suggest that we use this method to configure the network, the Network-Manager will make less errors than setting manually on the /e/n/interfaces files, and it can automatically switch when ehternet is connected.

## Configure WIFI (internet access) + Ethernet (share internet access to Dev board)

Here I want to keep laptop the internet access to the internet while at the same time, the laptop can also communicate with dev board through ethernet, and also make the dev board have the internet access.

### Why this is important ?

My dev board only has ehternet port and runs kernel linux system, so it does not support usb wifi yet, and since the new module needs to be cross compiled through my laptop, so I need the communication between the laptop and dev board (only ehternet works), and I don't want my laptop lose the internet access because of the ethernet connection. (In Ubuntu 12.04, when it detects that ehternet is connected, it will set the priority of eth0 higher than wlan0), so by default configuration, as soon as laptop has connected with dev board, the internet access is lost.
In windows, I can use the bridge connection mode, in this mode, the eth0 and wlan0 will be set in the same subnet and eth0 shares the internet access of wifi. Ubuntu 12.04 does not support this function by default. If I set manually the eth0 and wlan0 in the same subnet, only one connection (internet wlan0 access or dev board eth0 access) will be available at a time (depends on the value of the metric, you can check the value by typing "ip route show" in terminal).
So to achieve the goal above without using bridge mode, the subnet of wifi and the subnet of ethernet must be different. The easiest way is stil using netwwork manager to configure. 

### Network manager configuration
1. make sure that wlan0 has internet access
2. connect the laptop with dev board by ethernet (if necessary restart network manager to make sure that Wired connection is active)
2. Open "Network" application -> Options -> Tab IPv4 Settings
3. In "Method" list, choose option "Shared to other computers"
4. Then click save
5. Restart network manager (terminal: sudo service network-manager restart)

### Check eth0 configuration
``` bash
ip route show
```
Here is the automatically generated eth0 configuration on my laptop

``` bash
default via 192.168.1.254 dev wlan0  proto static 
10.42.0.0/24 dev eth0  proto kernel  scope link  src 10.42.0.1  metric 1 
169.254.0.0/16 dev wlan0  scope link  metric 1000 
192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.39  metric 2 
```

So the eth0 is assigned to 10.42.0.1, then on the other side, the dev board should set eth0 also in the same subnet and netmask. Here I assigned eth0 in dev board to 10.42.1.75 (here you should find another way to configure the dev board, I used serial port to configure it)

In /etc/network/interfaces in dev board, here is my configuration:

``` bash
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.42.0.75
netmask 255.255.255.0
gateway 10.42.0.1
```

Then restart the network interface in dev board
``` bash
$ sudo /etc/init.d/networking restart
```

Then it is done. Now laptop can talk to dev board (example: telnet 10.42.0.75), and the dev also have access to the internet, bingo !


``` bash
huangt@huangt-HP-ProBook-4431s:~$ telnet 10.42.0.75
Trying 10.42.0.75...
Connected to 10.42.0.75.
Escape character is '^]'.

 _____   _____   _____    _       _   __   _  __    __ 
|  ___| /  _   |  _    | |     | | |   | |    / / 
| |__   | | | | | |_| |  | |     | | |   | |   / /  
|  __|  | | | | |  _  /  | |     | | | |   |   }  {   
| |     | |_| | | |    | |___  | | | |   |  / /   
|_|     _____/ |_|  _ |_____| |_| |_|  _| /_/  _ 



ok335x login: root
root@ok335x:~# ping google.fr
PING google.fr (173.194.45.79): 56 data bytes
64 bytes from 173.194.45.79: seq=0 ttl=51 time=39.767 ms
^C
--- google.fr ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 39.767/39.767/39.767 ms
root@ok335x:~# 
```



