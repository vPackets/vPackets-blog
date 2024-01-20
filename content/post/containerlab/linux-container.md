---
title: "Lightweight Linux Containers in Containerlab" # Title of the blog post.
date: 2024-01-19T17:33:23-06:00 # Date of post creation.
description: "Deploy lightweight linux container to test your lab in containerlab." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/containerlab/linux/containerlab.png" # Sets featured image on blog post.
featureImageAlt: 'Containerlab Logo !' # Alternative text for featured image.
featureImageCap: 'Containerlab Rocks !' # Caption (optional).
thumbnail: "/images/containerlab/linux/containerlab.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - Containerlab
  - Networking
weight: 1
# comment: false # Disable comment if false.
---

When I was labbing back in the days on GNS3, all I needed to test my routing was another router. That networking device sole purpose was to ping accross the network. I just had to create a default route pointing to the upstream router and I could ping (if my lab was working.)

![GNS Lab](/images/containerlab/linux/gns3-lab.png)

Nowadays and with infrastructure as code, I don't want to spend time configuring IP addresses on routers and preserve as much resources as I can.
I have decided to use 2 containers that will be configured with the same IP addresses for every single lab: 192.0.2.100/24 and 203.0.113.100 . 

![Linux Containers on Containerlab](/images/containerlab/linux/containerlab-linux.png)

These 2 IP addresses belong to subnets reserved for documentation as mentionned in [RFC5737](https://datatracker.ietf.org/doc/html/rfc5737) 

I did it by creating two lightweight containers with a few network tools that are useful for my daily lab operations (Just realized I could add scapy to the container as well):

- [iputils](https://github.com/iputils/iputils) 
- [iperf3](https://github.com/esnet/iperf)
- [mytraceroute(mtr)](https://en.wikipedia.org/wiki/MTR_(software)) 
- [bind-tools](https://pkgs.alpinelinux.org/package/edge/main/x86/bind-tools)
- [netcat-openbsd](https://ftp.netbsd.org/pub/pkgsrc/current/pkgsrc/net/netcat-openbsd/index.html) 
- [curl](https://curl.se/) 
- [nmap](https://nmap.org/)

All of my labs will have these 2 containers so I can test various things using very little resources. The docker images are 38MB and booting them is almost instant.

Usually I leave Eth0 for management and Eth1 for the datapath of my actual lab.

![clab network and datapath](/images/containerlab/linux/containerlab-linux-clab-network.png)

Here are the dockerfiles.

```
FROM alpine:latest

# Label the image
LABEL maintainer="nicolas@vpackets.net"

# Update package list and install network tools
RUN apk add --no-cache bash \
    iputils \
    iperf3 \
    mtr \
    bind-tools \
    netcat-openbsd \
    curl \
    nmap

COPY network-config.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/network-config.sh
# Set the default command to launch when starting the container
# CMD ["/usr/local/bin/network-config.sh"]
CMD ["sh"]


```

When the image is being built, it will copy the network-config.sh that needs to be in the same folder as the Dockerfile (example for Container-ISP-01):

```
#!/bin/sh 


ip addr add 192.0.2.100/24 dev eth1
ip link set eth1 up
ip route add 203.0.113.0/24 via 192.0.2.1 dev eth1
```

and this is the network-config.sh script for the Container-ISP-02:

```
#!/bin/sh 


ip addr add 203.0.113.100/24 dev eth1
ip link set eth1 up
ip route add 192.0.2.0/24 via 203.0.113.1 dev eth1
```

These networking-script will be ran when the container will be launched, thanks Containerlab

You can pull the image directly from the docker hub registry here: 

```
docker pull vpackets/alpine-tools-containerlab-isp-01
docker pull vpackets/alpine-tools-containerlab-isp-02
```

Here is an example if you want to launch that linux container in a containerlab yaml topology:

```
name: eBGP-c8K

topology:
  nodes:
    linux01:
      kind: linux
      image: vpackets/alpine-tools-containerlab-isp-01:latest
      exec:
        - sh /usr/local/bin/network-config.sh

    linux02:
      kind: linux
      image: vpackets/alpine-tools-containerlab-isp-02:latest
      exec:
        - sh /usr/local/bin/network-config.sh

    Cisco8201-1:
      kind: cisco_c8000
      image: 8201-32fh_214:7.10.1

    Cisco8201-2: 
      kind: cisco_c8000
      image: 8201-32fh_214:7.10.1

  links:
    - endpoints: ["linux01:eth1", "Cisco8201-1:FH0_0_0_0"]   
    - endpoints: ["Cisco8201-1:FH0_0_0_1", "Cisco8201-2:FH0_0_0_1"]
    - endpoints: ["linux02:eth1", "Cisco8201-2:FH0_0_0_0"]
```


When you launch the lab and access the linux containers, this is what you can expect:

```
vpackets@srv-containerlab:/home/vpackets $ docker exec -it clab-eBGP-CSR-linux01 sh

/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
85: eth0@if86: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:14:14:04 brd ff:ff:ff:ff:ff:ff
    inet 172.20.20.4/24 brd 172.20.20.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2001:172:20:20::4/64 scope global flags 02
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe14:1404/64 scope link
       valid_lft forever preferred_lft forever
90: eth1@if89: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 9500 qdisc noqueue state UP
    link/ether aa:c1:ab:a4:ab:cc brd ff:ff:ff:ff:ff:ff
    inet 192.0.2.100/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fea4:abcc/64 scope link
       valid_lft forever preferred_lft forever
/ # ip route
default via 172.20.20.1 dev eth0
172.20.20.0/24 dev eth0 scope link  src 172.20.20.4
192.0.2.0/24 dev eth1 scope link  src 192.0.2.100
203.0.113.0/24 via 192.0.2.1 dev eth1
/ #
```


Containerlab will automatically assign an IP on eth0 with a range of 172.20.20.0/24 and the custom script will take care of the eth1 configuration.

That way, I am able to identify these routes and subnets in my lab pretty easily and know which route belong to which host. 

Happy labbing 

PS : You can find other useful containers [here](https://github.com/vPackets/containers)
