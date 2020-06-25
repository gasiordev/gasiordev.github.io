---
layout: post
title: "FreeBSD Jails quickstart"
author: "Mikolaj Gasior"
---

This is a quick post on how to get a jail running in FreeBSD. Without any
additional information, this could be found in FreeBSD documentation.

### Enable jail, pf and gateway
In `/etc/rc.conf`, ensure you have the following entries:
```
jail_enable="YES"
pf_enable="YES"
gateway_enable="YES"
```
These enable jails and services necessary to setup internet in them.

### security.jail.allow_raw_sockets
Type `sysctl security.jail.allow_raw_sockets` and check if it's `1`. Use
`sysctl security.jail.allow_raw_sockets=1` to change it.

### Network interface
Create a network interface for your jails:
```
ifconfig lo2 create
ifconfig lo2 up
```

### Create anchors in Packet Filter
Ensure you have the following lines in `/etc/pf.conf`:
```
nat-anchor "jail/*"
rdr-anchor "jail/*"
anchor "jail/*"
```

Restart pf or run `pfctl -f /etc/pf.conf` to load up new configuration.

Anchor is like a label. In later steps, we will add routing rules to an anchor
so it's easy just to flush the anchor.

### Obtain jail source
Create directory that will contain your jail files and download base of the
operating system:
```
mkdir /usr/jail/jail1
fetch http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.1-RELEASE/base.txz -o /usr/jail/jail1/base.txz
tar -xvf /usr/jail/jail1/base.txz /usr/jail/jail1/
```

### Create jail configuration
Put the following lines into `/etc/jail.jail1.conf`:
```
jail1 {
host.hostname = jail1;
ip4.addr = 192.168.13.37;
path = "/usr/jail/jail1";
exec.start = "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown";
mount.devfs;
allow.raw_sockets=1;
allow.sysvipc=1;
}
```

### Bring up IP address
`192.168.13.37` is used in this post.
```
ifconfig lo2 inet 192.168.13.37/24 alias
```

### Allow jail access the internet
Create `/etc/pf.jail1.conf` and put the following contents (change `igb0` to
your gateway network interface).
```
nat pass on igb0 from 192.168.13.37/32 to any -> (igb0:0)

```
Use `pfctl` to add the rule to an anchor:
```
pfctl -a jail/jail1 -f /etc/pf.jail1.conf
```

#### Port forwarding
If you'd like to forward a port on your external interface to a port
in jail, use line similar to below (change ports and put it in `pf.jail1.conf`):
```
rdr pass on igb0 inet proto tcp from any to (igb0:0) port 8222 -> 192.168.13.37 port 22
```

### Finally start the jail
Just run the following:
```
jail -c -f /etc/jail.jail1.conf
```

(You might as well use `service jail start jail1` - it should read the config
file).

Type `jls` to check if your jail is running.

### Stop jail
Command `jail -r jail1` will stop your jail. If you'd like to destroy it, remove
its source directory of `/usr/jail/jail1`. If it's impossible then run `mount`
to check if some `dev` isn't still mounted. If so, use `umount` on it.

## Bgos
If you feel this is too much hassle, feel free to use a tiny tool I have created
recently called Bgos. It simplifies running jails. It can be found
[here](https://github.com/gasiordev/bgos).
