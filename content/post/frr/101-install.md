+++
title = "FRR 101 - Install"
description = "FRR 101 and basic connectivity test"
date = "2023-03-13"

weight = 2

tags = [
    "FRR",
    "Linux Networking"
]

categories = [
    "Networking",
]

thumbnail = "images/router.png"
+++

Networking vendors like Cisco, Arista , Juniper have developped their own operating system and all networking engineers had to learn them in order to be relevant when designing, implementing or supporting their own network.

Each of these operating systems has its unique features and capabilities, and they are used in different network environments and for different purposes. Some are more suitable for small-scale networks, while others are designed for large-scale enterprise networks.
It should be noted that most of these Network OS are based on Linux or BSD. 

Cisco even has more than one OS for their platforms:

Cisco CatOS - Deprecated.
Cisco IOS - used for Cisco routers and switches
Cisco IOS XE - used for Cisco Catalyst 9000 series switches
Cisco IOS XR - used for Cisco ASR 9000 series routers
Cisco NX-OS - used for Cisco Nexus series switches


It is important to fathom that there are some open source projects focusing on network operating systems (NetOS):

* [Quagga](https://en.wikipedia.org/wiki/Quagga_(software)) - open-source routing suite that supports multiple routing protocols. Maintained between 2001 and 2018 and created as an alternative to proprietary routing solutions. It was based on the Zebra routing suite.

* [FRR](https://frrouting.org/) - Fork and successor to Quagga, this routing daemon is widely used in the industry by networking vendors like VMware (for their NSX Edge network stack) and Nvidia Cumulus. It is designed to be more modular and flexible than Quagga, making it easier to add new features and support for new routing protocols.

* [Bird](https://gitlab.nic.cz/labs/bird) - In their own terms: The BIRD project aims to develop a dynamic IP routing daemon with full support of all modern routing protocols, easy to use configuration interface andpowerful route filtering language, primarily targeted on (but not limited to) Linux and other UNIX-like systems and distributed under the GNU General Public License. I have never used BIRD but will definitely spend more time doing so as it supports an extensive range of protocols and is actively developped. It is probably the oldest open source networking project still in activity. Fun fact : BIRD is the most used route server amongst European Internet exchanges so I guess it has been build with scalability in mind :). 

* [GoBGP](https://osrg.github.io/gobgp/) - Routing daemon focusing solely on the BGP routing protocol and hence its name, written in Go./

If you want to compare these network stacks, [Justin Pietsch did an amazing job here](https://elegantnetwork.github.io/posts/followup-measuring-BGP-stacks/)

I have been using FRR extensively in my career and will spend some time showing the particularities of this network stack


## What is FRR ?

Designed to provide a full-featured routing stack for Linux-based network operating systems. It is developed by a group of networking experts and is distributed under the GNU General Public License version 2, which makes it **freely** available to anyone.

It supports everything you need (well...) in terms of routing protocols like OSPF, BGP, IS-IS and even multicast. As I mentionned previously, it is widely used by networking vendors making it suitable for large-scale datacenters. 

VTYSH is an integrated shell for Quagga/FRR that allows you to configure the network interfaces and routing protocol the same way you would do in a traditionnal switches. Network engineers used to the Cisco IOS language will be able to navigate easily through VTYSH making it easy to adopt by reducing the learning curve. VTYSH comes automatically with FRR.

Back to the FRR architecture, it uses separate daemons that are independent and responsible for a specific functionality or routing protocol. It allows you to implement only the daemon you need in your architecture without having other dependencies 



## Basic topology used

We will install FRR on 2 Ubuntu server 22.04 virtual machines.

Each virtual machine will be deployed with at least 2 network interfaces cards :

  - ENS160 will be used as a management interface
  - ENS192 will be configured using FRR. This interface on vCenter should use a dedicated VLAN and shared between the two ubuntu servers

![FRR basic topology](/images/frr/basic-install/FRR-basic-topology.png)


## FRR installation in Ubuntu server 22.04

We will be using this [documentation related to FRR for debian packages](https://deb.frrouting.org).

After Ubuntu 22.04 is installed, we need to download the package informations from all configured sources.

``` 
sudo apt-get update
sudo apt-get upgrade  => never hurts anyone to upgrade to the new packages :) 
```

Now we need to add the GPG key for the package source into the apt keyring of Ubuntu.

```
curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -
```

Then we need to add the FRR package repository to the list of package sources for apt.

```

FRRVER="frr-stable"
echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list

```

Using the FRRVER environment , you can choose which version of FRR you want to install. 
Then install the FRR package and its dependencies:

```
sudo apt update && sudo apt install frr frr-pythontools
```

After this, we need to check if frr is running:

![FRR status](/images/frr/basic-install/FRR-status.png)


We can now start configuring our interfaces and test the connectivity.

Let's have a look at the interfaces : 

``` bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether ****** brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet ** brd ** scope global ens160
       valid_lft forever preferred_lft forever
    inet6 ** scope link
       valid_lft forever preferred_lft forever
3: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether ****** brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet6 ****** scope link
       valid_lft forever preferred_lft forever
4: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:bf:07:c0 brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet6 fe80::250:56ff:febf:7c0/64 scope link
       valid_lft forever preferred_lft forever
```

We can see that ens192 has no interfaces configured, let's use the FRR VTYSH shell to configure it on the frr server. (you will need to perform this on both ubuntu servers using different IP addresses of course)

```
nmichel@srv-frr-02:~$ sudo vtysh
Hello, this is FRRouting (version 8.4.2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

srv-frr-02# config t
srv-frr-02(config)# int ens192
srv-frr-02(config-if)# ip add 10.1.1.2/24
srv-frr-02(config-if)# no shut
srv-frr-02(config-if)# end
srv-frr-02#
```

Let's do some verification:


```
nmichel@srv-frr-02:~$ ip addr show ens192
3: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:bf:3c:df brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 10.1.1.2/24 brd 10.1.1.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:febf:3cdf/64 scope link
       valid_lft forever preferred_lft forever
nmichel@srv-frr-02:~$
```

Since we added another interfaces, the Linux routing table has been udpated:

```
nmichel@srv-frr-02:~$ ip route
default via 192.168.199.1 dev ens160 proto static
10.1.1.0/24 dev ens192 proto kernel scope link src 10.1.1.2
192.168.199.23/24 dev ens160 proto kernel scope link src 192.168.199.23
```

Connectivity tests can now occur:

```
nmichel@srv-frr-02:~$ ping 10.1.1.1 -c 1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.226 ms

--- 10.1.1.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.226/0.226/0.226/0.000 ms
```

In the next blogposts, I will demonstrate how to create and automate a full BGP configuration using FRR.

