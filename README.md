## Overview

The bandwidth available over local DSL lines isn't very high in the 
area we live in so we've had to use several techniques to increase the 
overall capacity of our internet access and one approach has been to use 
3G/4G connections

## Quick Start

It *should* just be a case of copying all of the files under `etc` into their
matching locations on running/working Raspberry Pi, reboot, and everything
hopefully will work :D


## Hardware

These are the parts I used

* [Raspberry Pi 2 Model B](https://www.amazon.co.uk/gp/product/B00T2U7R7I/ref=ppx_od_dt_b_asin_title_s00?ie=UTF8&psc=1)
* [Huawei E3372-LTE](https://www.amazon.co.uk/gp/product/B0104LV06M/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
* USB power supply for the Raspberry Pi 2A capacity
* 4G SIM card from THree - they provide various price plans including 'unlimited' data (it's capped at around 475Gb per month)

You could certainly use a more up-to-date Raspberry Pi device and even the 
PoE HAT for network/power.

The Raspberry Pi is running Raspbian 9.4 (stretch) Lite - the version that 
doesn't include a desktop/GUI.


## Topology

### Connections

```
 _________________          ______________       ________________
|                 |        |              |     |                |
| Firewall/Router |--------| Raspberry Pi |-----| USB LTE Dongle |
|_________________|        |______________|     |________________|

```

### Networks

#### Management

SSH access to the Raspberry Pi is over `192.168.123.0/24` 

#### Routing

In order to keep traffic for routing over the LTE link seperate from the 
management network a VLAN (10) is used on `192.168.130.0/24`

The Firewall/Router that is sending packets to the Raspberry Pi has an 
address `192.168.130.4` on the VLAN 10 network.


#### USB Network

The E3372 device can be in two modes, "stick" or "HiLink".  You can 
apparently [re-flash to force stick mode](https://blog.le-vert.net/?p=196)
but I did not even attempt this and would not recommend it.

The E3372 devices I have are all in "HiLink" mode which seems to mean that
the USB dongle appears as a network (ethernet) device, it uses the 
address `192.168.8.1`

The ethernet address for the Raspberry Pi side of this network has been set 
to `192.168.8.10`


## Setup

### Operation

The basic premise of this setup is that Linux allows different routing
tables to be used when, in this case, traffic is coming from a specific
source address.

Essentially the Raspberry Pi device is using two routing tables, 

* default - management traffic goes via the main firewall/router
* three - traffic on VLAN10 goes over the USB dongle and 4G/LTE network


### Routing tables

Added a table called `three`, see the `etc/iproute2/rt_tables` file

### Devices and routing rules

See `etc/network/interfaces.d` for the config for each network device

* eth0 is the main Raspberry Pi ethernet port
* eth1 is the USB ethernet network from the dongle
* vlan10 is the network over traffic is sent to be routed over 4G/LTE

### Firewall NAT

Traffic from VLAN 10 must use SNAT to make packets appear to originate from 
the Raspberry Pi end of the USB dongle network.

See the `nat` table rules in `etc/iptables/rules.v4`


### Sysctl

NOTE: DO NOT FORGET THIS!

The forwarding of traffic needs to be enabled in the kernel otherwise none 
of the above works at all.  I kept forgeting to do this!

See the file under `etc/sysctl.d`



## References

* [ip rule man page](http://man7.org/linux/man-pages/man8/ip-rule.8.html)
* [ip forwarding](https://www.systutorials.com/1372/setting-up-gateway-using-iptables-and-route-on-linux/) 
* [NAT](https://netfilter.org/documentation/HOWTO/NAT-HOWTO-6.html)
