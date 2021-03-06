---
layout: post
title: Install L2TP/IPSEC/PPTP Server On Ubuntu(14.04) Server
category: idev
tags: linux ubuntu l2tp pptp ipsec
year: 2014
month: 06
day: 15
published: true
customid: 20140615_ubuntu_l2tp
summary: How to install L2TP/IPSEC/PPTP Server on Ubuntu Server (14.04) and configure for usage.
image: idev/ubuntu-l2tp-ipsec.png
---

I have a [Linode](https://www.linode.com/?r=99aaa797e467227583c245023b31dbaf9d195227) VPS with 2TB monthly  bandwidth limitation for daily Linux and other usage. This Guide will walk you through the process of installing a L2TP VPN Server on Ubuntu Server 14.04.

1. Install `xl2tpd`, `openswan` and `ppp`:

```bash
sudo apt-get --no-install-recommends install xl2tpd openswan ppp pptpd
```

2. After all apps installed we need to conifuguration the system and apps to make all services run well as we wish. First, we need to set the firewall and make the kernel forward the ip packet.
For l2tp/ipsec:

```bash
iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o eth0 -j MASQUERADE
```

 For pptp:

```bash
iptables -A FORWARD -p tcp --syn -s 172.16.0.0/16 -j TCPMSS --set-mss 1356
```

`eth0` is the network interface with public ip.
`172.16.0.0/16` is the ip range vpn we want the client to use when connected. You can use any [private network ip](http://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces) address you want. Remember to make this different from your local network ip address.

3. Second, we will disable `accept_redirects` and `send_redirects` to disable the kernel ICP redirects:

```bash
    for each in /proc/sys/net/ipv4/conf/*
        do
            echo 0 > $each/accept_redirects
            echo 0 > $each/send_redirects
        done
```

4. Finally, we will enable the ip packet forwarding and disable the kernel ICP redirects for boot by adding the following lines at the end of `/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.icmp_ignore_bogus_error_responses = 1
```

Note: We can create a customized init script to do the 1st and 2nd steps automatically with the following steps:
1). Create a customized service script `/etc/init.d/vpn`(You can use any file name here):

```bash
#!/bin/sh
# Customized startup script for L2TP/IPSec, PPTP
### BEGIN INIT INFO
# Provides:          http://hdi.ly
# Required-Start:    $network $remote_fs $syslog $named
# Required-Stop:     $syslog $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start Openswan IPsec and pptpd at boot time
# Description:       Customized start script to start ipsec, xl2tpd and pptpd
### END INIT INFO

case "$1" in
    start)
    echo "Starting L2TP/Ipsec,PPTP VPN Service"
    iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o eth0 -j MASQUERADE
    iptables -A FORWARD -p tcp --syn -s 172.16.0.0/16 -j TCPMSS --set-mss 1356
    for each in /proc/sys/net/ipv4/conf/*
        do
            echo 0 > $each/accept_redirects
            echo 0 > $each/send_redirects
        done
    /etc/init.d/ipsec start
    /etc/init.d/xl2tpd start
    /etc/init.d/pptpd start
    ;;

    stop)
    echo "Stopping L2TP/Ipsec,PPTP VPN Service"
    iptables --table nat --flush
    /etc/init.d/ipsec stop
    /etc/init.d/xl2tpd stop
    /etc/init.d/pptpd stop
    ;;
    restart)
    echo "Restarting L2TP/Ipsec,PPTP VPN Service"
    iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o eth0 -j MASQUERADE
    iptables -A FORWARD -p tcp --syn -s 172.16.0.0/16 -j TCPMSS --set-mss 1356
    for each in /proc/sys/net/ipv4/conf/*
        do
            echo 0 > $each/accept_redirects
            echo 0 > $each/send_redirects
        done
    /etc/init.d/ipsec restart
    /etc/init.d/xl2tpd restart    
    /etc/init.d/pptpd restart
    ;;

    *)
    echo "Usage: /etc/init.d/vpn  {start|stop|restart}"
    exit 1
    ;;
esac
```

This script will update the firewall settings when you startup the `ipsec`, `xl2tpd` and `pptpd` services. You need add executive permission to this file:

```bash
sudo chmod 755 vpn
```

2). And then replace the default `ipsec`, `xl2tpd` and `pptpd` init script with this customized script:

```bash
update-rc.d -f ipsec remove
update-rc.d -f xl2tpd remove
update-rd.d -f pptpd remove
update-rc.d vpn defaults
```

5. Now we can configure `pptpd`, `xl2tpd`, and `p2tpd` separately. For `pptp`, we just need to configure the `/etc/pptpd.conf` with the getway IP and client IP range:

```bash
localip 172.16.2.1
remoteip 172.16.2.2-254
```

`localip`: Server IP, served as the pptp server gateway IP.
`remoteip`: The IP range server assigned when an client connected, must be an subset of the IP range configured in the iptabs configuration section.

6. Configure `ipsec` by editting the `/etc/ipsec.conf` configuration file. Here is an entire configuration template, just replace the `***.***.***.***` with your server IP.

```bash
# /etc/ipsec.conf - Openswan IPsec configuration file

# This file:  /usr/share/doc/openswan/ipsec.conf-sample
#
# Manual:     ipsec.conf.5


version 2.0 # conforms to second version of ipsec.conf specification

# basic configuration
config setup
    # Do not set debug options to debug configuration issues!
    # plutodebug / klipsdebug = "all", "none" or a combation from below:
    # "raw crypt parsing emitting control klips pfkey natt x509 dpd private"
    # eg:
    # plutodebug="control parsing"
    # Again: only enable plutodebug or klipsdebug when asked by a developer
    #
    # enable to get logs per-peer
    # plutoopts="--perpeerlog"
    #
    # Enable core dumps (might require system changes, like ulimit -C)
    # This is required for abrtd to work properly
    # Note: incorrect SElinux policies might prevent pluto writing the core
    dumpdir=/var/run/pluto/
    #
    # NAT-TRAVERSAL support, see README.NAT-Traversal
    nat_traversal=yes
    # exclude networks used on server side by adding %v4:!a.b.c.0/24
    # It seems that T-Mobile in the US and Rogers/Fido in Canada are
    # using 25/8 as "private" address space on their 3G network.
    # This range has not been announced via BGP (at least upto 2010-12-21)
    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v4:25.0.0.0/8,%v6:fd00::/8,%v6:fe80::/10
    # OE is now off by default. Uncomment and change to on, to enable.
    oe=off
    # which IPsec stack to use. auto will try netkey, then klips then mast
    protostack=auto
    # Use this to log to a file, or disable logging on embedded systems (like openwrt)
    #plutostderrlog=/dev/null


conn L2TP-PSK-noNAT
        authby=secret
        #shared secret. Use rsasig for certificates.

        pfs=no
        #Disable pfs

        auto=add
        #the ipsec tunnel should be started and routes created when the ipsec daemon itself starts.

        keyingtries=3
        #Only negotiate a conn. 3 times.

        ikelifetime=8h
        keylife=1h

        ike=aes256-sha1,aes128-sha1,3des-sha1
        phase2alg=aes256-sha1,aes128-sha1,3des-sha1
        # https://lists.openswan.org/pipermail/users/2014-April/022947.html
        # specifies the phase 1 encryption scheme, the hashing algorithm, and the diffie-hellman group. The modp1024 is for Diffie-Hellman 2. Why 'modp' instead of dh? DH2 is a 1028 bit encryption algorithm that modulo's a prime number, e.g. modp1028. See RFC 5114 for details or the wiki page on diffie hellmann, if interested.

        type=transport
        #because we use l2tp as tunnel protocol

        left=***.***.***.***
        #fill in server IP above

        leftprotoport=17/1701
        right=%any
        rightprotoport=17/%any

        dpddelay=10
        # Dead Peer Dectection (RFC 3706) keepalives delay
        dpdtimeout=20
        #  length of time (in seconds) we will idle without hearing either an R_U_THERE poll from our peer, or an R_U_THERE_ACK reply.
        dpdaction=clear
        # When a DPD enabled peer is declared dead, what action should be taken. clear means the eroute and SA with both be cleared.
```

7. Configure the `ipsec` pre-shared key, the configuration file is `/etc/ipsec.secrets`:

```bash
# This file holds shared secrets or RSA private keys for inter-Pluto
# authentication.  See ipsec_pluto(8) manpage, and HTML documentation.

# RSA private key for this host, authenticating it to any other host which knows
# the public part.  Suitable public keys, for ipsec.conf, DNS, or configuration
# of other implementations, can be extracted conveniently with "ipsec
# showhostkey".

# this file is managed with debconf and will contain the automatically created RSA keys
# include /var/lib/openswan/ipsec.secrets.inc
***.***.***.***  %any:   PSK     "some-psk-string"
```

In the configuration file, `****.***.***.***` is the server public IP and `some-psk-string` is your customized psk.

9. Configure xl2tpd, use your favorite editor to edit the `/etc/xl2tpd/xl2tpd.conf` file by repacing all the content with the following lines:

```bash
[global]
ipsec saref = yes
saref refinfo = 30

;debug avp = yes
;debug network = yes
;debug state = yes
;debug tunnel = yes

[lns default]
ip range = 172.16.1.30-172.16.1.100
local ip = 172.16.1.1
refuse pap = yes
require authentication = yes
;ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

10. Configure ppp by replacing the content of `/etc/ppp/options.xl2tpd` with the following lines with your favorite editor:

```bash
require-mschap-v2
ms-dns 8.8.8.8
ms-dns 8.8.4.4
auth
mtu 1200
mru 1000
crtscts
hide-password
modem
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
```

11. Add user, all the users are defied in `/etc/ppp/chap-secrets`. Here is an example configuration:

```bash
# Secrets for authentication using CHAP
# client    server  secret          IP addresses
user  *   passwd  *
```
 `user` = username for the user.
 `server` = the name we define in the ppp.options file for xl2tpd.
 `passwd` = password for the user.
 `IP Addresses` = leave to * for any address or define addresses from were a user can login.

12. All done, now start and test the service

```bash
sudo service vpn restart 
```

If every thing goes well, you will see the following outputs

```bash
Restarting L2TP/Ipsec,PPTP VPN Service
ipsec_setup: Stopping Openswan IPsec...
ipsec_setup: Starting Openswan IPsec U2.6.38/K3.15.4-x86_64 ...
Restarting xl2tpd: xl2tpd.
 * Restarting PoPToP Point to Point Tunneling Server pptpd               [ OK ]
```
