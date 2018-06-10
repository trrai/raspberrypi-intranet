# raspberrypi-intranet
![raspberrypi logo](https://i.imgur.com/f2goO4F.jpg)
## Overview
For this final project, we made use of all four of our raspberry pi’s to create an internal network which consisted of one ISP router pi and three pi edge networks. The three edge network pi’s connected physically into the ISP pi, while our computers connected to our pi’s via wireless ssh through the wlan0 port on each pi. With this network set-up, the ISP pi was able to provide internet access to the three edge networks, which acted like wireless access points with no actual access to the internet without connection to the ISP pi. The three edge network pi’s hosted mail servers which allowed for email communication between pi’s and acted as a visual way for us to see that all our configurations were working properly and our pi’s were able to communicate.


Our internal network was then able to connect to the class network where our pi’s were able to communicate with other group’s pi’s. This allowed for us to send and receive email between all clients on the classes network and essentially expanded our network to be able to interact with other group’s subnetworks. We were able to accomplish peer to peer communication through use of Free Range Routing and BGP. FRR was identical to the Packet Tracer practice that all of us had completed throughout the quarter.

Each member of our group also had a personal DNS zone related to NBA players. The three edge zones were harden.pi, kobe.pi, and shaq.pi. The ISP zone was called nba.pi. Each zone had its own authoritative server (ns1.ZONE_NAME.pi) as well as a unique authoritative server IP. This allowed for each private mail server to communicate through the ISP pi.
Obstacles that we ran into while configuring each of our pi’s are specified in the sections below but a general overview of troubleshooting encapsulates issues with BGP routing setup and the mail servers. The most critical issue we ran into was an error while setting up the routers in FRR which denied route sharing to eth2, and eth3. This was difficult to troubleshoot because the pi’s were assigned on a different ethX port on each startup so we assumed the setup was related to the edge networks, not the ISP. In addition to this, our group was forced to rebuild and restart the dovecot, postfix, and rainloop Docker containers several times while attempting to remove bugs with user authentication and cross peer communication. Holistically there was over 20 to 25 hours of solely debugging as a group to overcome some of these configuration issues, with new problems seeming to appear one after another.

## Schema
![Image of our network schema](https://image.ibb.co/jCPGHo/INFO341_Network.png)

## Specifications 

### ISP Pi 
ISP Router Features:
- Internet Access
- BGP peering and FRR
- IPTable rules to allow forwarding
- Communication between hosts
- Connection to class network (eth0)
- DNS subdomain for internal and external network

The ISP pi was set up so that port eth0 was used for connecting to the main external network and eth1, eth2, and eth3 were ports used by the three edge pi’s. This device then pulled internet connection and BGP routes from the eth0 to be shared with the hosts. With the BGP routing in place, peers on the network were able to ping each other and access zones configured with bind9.

### Edge Pi
Edge Network Features:
- WAP (Internet access when connected to the ISP pi)
- DHCP (eth0)
- DNS Caching Server
- Firewall
- Private Mail Server
- BGP peering and FRR

Each edge network used wlan0 as an access point for wireless ssh and communication, while eth0 was used to connect to the ISP. Since each edge was a wireless access point, anybody with a password could connect to an edge pi and use the internet or connect to any pi on the network via ssh. Each edge firewall allowed public facing traffic to come into the network (for example, DNS, SMTP, or webmail) and also allowed forwarding if it was related to any sessions started by a LAN on the network. Each edge also contained public and private DNS zones for internal use within our network as well as external use for public search queries.

## Individual Configuration Summaries
### Bogdan's Pi
My pi acted as the primary mail server for the group and as a WAP. My pi was set up as an internal network and connected through Tejveer’s ISP pi through my eth0. My wlan0 handled wireless communication and allowed my to SSH into my pi wireless. In this way my pi was set up as a Wireless Access Point getting internet from the ISP pi. Without the ISP, I was able to use internet sharing tool on my MAC to share internet to my eth0 and access the internet without connection to an ISP router. I also separated my network into a virtual interface by partitioning IPs above 172.23.20 on the dummy0 interface. Using DNS zones and docker containers (dovecot, rainloop, and postfix), I was able to configure a private mail server for harden.pi and corp.harden.pi on SMTP and IMAP and could connect to it through my browser using rainloop.

### Tejveer's Pi
My personal responsibilities for the group involved setting up the configurations to allow connected peers on the USB ports to communicate. These USB ports were then mapped to the eth1, eth2, and eth3 interfaces on my pi’s configurations. The ethernet port on this pi maps to eth0 and was reserved to be connected to the class network for the final schema. In order to separate the network into virtual internal and external partitions, we used a dummy0 interfaced and assigned ip’s above X.X.X.129 for that domain. Containers ran on the pi included dotpi, dovecot, postfix, and rainloop. Dotpi is the top level domain meant to communicate with the rest of the class on the 10.10.10.10 address and share zones. Dovecot, postfix and rainloop work in communion to run a functional mail server on SMTP and IMAP.

### Jordan's Pi
My Pi functioned as a network, its role was quite simple. I received internet directly from Tejveer’s Pi(the ISP) through ethernet - this was eth0 on my interfaces. This internet was fed into the wlan0 network setup on the device. The pi acted as a wireless router, under the SSID of kobe with an IP of 172.23.23.1, where devices such as my personal computer could connect to and use the internet as well as ssh into my pi or other pis on the network. The IPtables on the pi was setup to allow traffic from dns, mail, http etc. to come through the pi. Mail Servers were setup through docker containers of: dovecot, rainloop, and postfix - through these setups we were able to authenticate email addresses and communicate over the network and onto the host pi. Like the ISP we separated the pi into different partitions - mine was called dummy0 which had an ipaddress of 172.23.23.130.

### Keegan's Pi
My goal of this project was setting my pi as edge network to perform as a wireless access point. My pi was physically connected to Tejveer’s pi(ISP Pi) and configured as eth0 in interface, ultimately, any user with access to pi could connect to pi through ssh and connect on the network. Firewall configuration/ IPtables allowed and denied trafficking coming through from DNS and forwarding if it was on under appropriate setting. My personal pi was configured with public/private DNS zones for internal networking. Setting up/configuring BGP through vtysh command allowed to function as wireless access point/wireless router. Unfortunately, I was unable to set up docker container due to fail to troubleshoot wget function.

## Main Obstacles
### Bogdan's Pi
- BGP configuration erased in FRR settings on multiple occasions
  - Most likely cause is a low voltage error. When there was a low voltage
error, BGP probably failed to load and frr.conf seemed to have resorted to an older save of the file. There is no way to see that the BGP configuration was reset.
  - How I got around the issue: I would advise that if somebody’s pi suddenly loses internet seemingly out of nowhere then they should check ​journal -xe​. ​If one or multiple low voltage errors are present, then I would advise opening the FRR configuration in ​vtysh​ and typing show run​, then check to see if all previously created BGP configuration is up to date. If not rewrite the BGP settings and save the configurations with the command c​ opy run start ​at least three times to be safe.
- Eduroam issues with internet sharing and wifi configuration
  - The cause seems to be since eduroam is a private educational wireless
network, they have increased security settings. This makes it more challenging to do more diverse things with the wifi like internet sharing, connecting a raspberry pi, etc.
  - Solution: use the University of Washington wifi.
  
### Tejveer's Pi
- Misspelling in db.nba.pi zone file
  - After a long process of changing configurations and restarting bind9
repeatedly, my dns zone issues boiled down to a missing period
following the end of my zone name while declaring the NS record
- Missing eth2 and eth3 in interfaces.d + dhcpcd.conf
  - During initial configuration of the ISP, we established the interface configurations for eth1 but held off on creating the files for the other ports. This came back to be a large problem when we connected more than one pi and the interfaces would not come up
- FRR missing eth2 and eth3 configurations
  - When trying to discover why routes were only being shared for one of
the peers, it ended up being an issue with only eth1 being declared in the vtysh console. After seeing the running configuration, we were able to add the remaining interfaces

### Jordan's Pi
- Issues with eth0 interface
  - Had auto-hotplug instead of allow-hotplug which caused a whole
variety of problems and after several hours of troubleshooting and staring at the files - Davin eventually found the mistake and things were fixed - this allowed internet being fed from tejveers pi into the device
- Not swapping eth0 with wlan0 - causing ssh issues
  - The issue occured because internet was not being fed in the direction i had set up through the interfaces - however upon looking again at the ip tables, my previous setup seemed to have reverted showing each eth0/wlan0 being swapped preventing the wireless router to function properly.
- Docker authentication errors
  - Each of my containers seemed to be started correctly with correct
users. However upon logging in the user was able to setup the email, create a new message, but upon sending would indicate “user authentication error”. We initially thought it was due to my address not being added to the TLD, but this did not seem to be the case - this issue still occurs and I am still unable to send emails from the interface.

### Keegan's Pi
- Unable to set up Docker containers
  - wget command was not working although pi was connected to the internet
- BGP configuration
  - When I was done configuring BGP through vtysh, although I got my ip
through Tej’s pi, I did not have access to the internet.
  - Solution: I used j​ ournal -xe​ command to troubleshoot, we found there
were couple misspelling in configuration.
- Ssh issue
  - Once I configured BGP I was unable to access my pi
  - Solution: We connected the pi to the screen and we found couple typo
on ip tables and ​dhcpcd.conf​ which denied the access.
