---
layout: post
title: "Linux VM on bhyve networking using bridge"
author: "Mikolaj Gasior"
---

There are many great articles on running Linux VM in FreeBSD using bhyve. However,
you might get into trouble when setting internet access for the VM. I thought I'll
post a quick snapshot of my configuration.

### Solution
I have one network interface `igb0` which has public IP address, `tap0` interface
running Linux VM and `bridge0` interface that links both. In addition to that,
PF `nat pass` rule is used to give VM internet access. 
Bridge has an address of 192.168.30.1 and VM - 192.168.30.31.


### Setting up
On the FreeBSD hosts:
```
ifconfig tap0 create up
ifconfig bridge0 create up
ifconfig bridge0 inet 192.168.30.1
```

To add a NAT rule using PF, edit you pf.conf file (`/etc/pf.conf`?),
add the following line:
```
nat pass on igb0 from 192.168.30.0/24 to any -> (igb0:0)
```
Use `pfctl -f /etc/pf.conf` to flush all existing rules and load new ones from
the file.

Once VM is started, `tap0` interface should change its `status` to `active` and
a line `Opened by PID XXXX` should appear in `ifconfig` output (see CLI output).

On the Linux VM, we set IP address for eth device, set the gateway to bridge
IP address and add a nameserver:
```
ifconfig enp0s2 inet 192.168.30.31
route add default gw 192.168.30.1
cat >/etc/resolv.conf <<EOF
nameserver 8.8.8.8
EOF
```


### CLI output
#### FreeBSD host
`ifconfig` prints out the following interfaces:
```
igb0: flags=8943<UP,BROADCAST,RUNNING,PROMISC,SIMPLEX,MULTICAST> metric 0 mtu 1500
	options=a520b9<RXCSUM,VLAN_MTU,VLAN_HWTAGGING,JUMBO_MTU,VLAN_HWCSUM,WOL_MAGIC,VLAN_HWFILTER,VLAN_HWTSO,RXCSUM_IPV6>
	ether xx:xx:xx:xx:xx:xx
	inet XXX.XXX.XXX.XXX netmask 0xffffff00 broadcast AAA.AAA.AAA.255
	inet6 CCCC::CCCC:CCC:CCCC:CCCC%igb0 prefixlen 64 scopeid 0x1
	inet6 BBBB:BBBB:BBB:BB:: prefixlen 64
	media: Ethernet autoselect (1000baseT <full-duplex>)
	status: active
	nd6 options=8063<PERFORMNUD,ACCEPT_RTADV,AUTO_LINKLOCAL,NO_RADR,DEFAULTIF>
tap0: flags=8943<UP,BROADCAST,RUNNING,PROMISC,SIMPLEX,MULTICAST> metric 0 mtu 1500
	options=80000<LINKSTATE>
	ether xx:xx:xx:xx:xx:xx
	groups: tap
	media: Ethernet autoselect
	status: active
	nd6 options=43<PERFORMNUD,ACCEPT_RTADV,NO_RADR>
	Opened by PID 83388
bridge0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
	ether 02:bd:8f:27:42:00
	inet 192.168.30.1 netmask 0xffffff00 broadcast 192.168.30.255
	id 00:00:00:00:00:00 priority 32768 hellotime 2 fwddelay 15
	maxage 20 holdcnt 6 proto rstp maxaddr 2000 timeout 1200
	root id 00:00:00:00:00:00 priority 32768 ifcost 0 port 0
	member: tap0 flags=143<LEARNING,DISCOVER,AUTOEDGE,AUTOPTP>
	        ifmaxaddr 0 port 4 priority 128 path cost 2000000
	member: igb0 flags=143<LEARNING,DISCOVER,AUTOEDGE,AUTOPTP>
	        ifmaxaddr 0 port 1 priority 128 path cost 20000
	groups: bridge
	nd6 options=41<PERFORMNUD,NO_RADR>
```

And, PF (`pfctl -s all`) has the following rule:
```
nat pass on igb0 from 192.168.30.0/24 to any -> (igb0:0)
```

#### Linux VM
`ifconfig` and `route` on my Linux virtual machine are the following.
```
enp0s2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.31  netmask 255.255.255.0  broadcast 192.168.30.255
        inet6 fe80::2a0:98ff:fe27:a718  prefixlen 64  scopeid 0x20<link>
        ether 00:a0:98:27:a7:18  txqueuelen 1000  (Ethernet)
        RX packets 499105  bytes 341616826 (341.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14323  bytes 1105761 (1.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 enp0s2
192.168.30.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s2
```

