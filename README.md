# raspberrypi-intranet
## Summary
Repository meant to store the configurations and image files for the raspberry Pi devices joined to created a private and public internet running mail servers.

## Contents
Each branch on the repo represents a router that was connected to our network environment and their associated configurations on the Pi. Tej's branch acts as the Internet Service Provider with Bogdan, Jordan and Keegan's Pi's acting as the edge networks serving clients. Outside users connected to any of these networks through WLAN gain access to the web of nameserver records and hosted mail server that allows messages to be sent from Pi to Pi. This mail server functionality was accessed through a Docker container running on each of the devices and communicated via dovecot, postfix and rainloop making use of IMAP and SMTP. 

## Schema
![Image of our network schema](https://image.ibb.co/jCPGHo/INFO341_Network.png)
