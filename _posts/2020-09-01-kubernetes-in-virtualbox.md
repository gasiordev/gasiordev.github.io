---
layout: post
title: "Kubernetes in VirtualBox"
author: "Mikolaj Gasior"
---

When running Kubernetes cluster on local computer with master and worker nodes
provided by VirtualBox VM's, I encountered problems with getting network to work.
So here's a quick snapshot of steps to get it working.

## VirtualBox
NAT interface with IP address of 10.0.2.15.

Master and 3 worker nodes have host-only interfaces with IP addresses of:
192.168.30.30, 192.168.30.31, 192.168.30.32, 192.168.30.33.

My pod network CIDR is 192.168.0.0/16.

For k8s networking, Calico 3.16.1 was used. 

## Commands used to create the cluster

```
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.30.30 --control-plane-endpoint=192.168.30.30 --upload-certs
```

The following line was added on all of the nodes (IP address needs to match
node's IP address, so .30, .31, ...):
```
root@km:/home/vagrant# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf  | grep KUBELET_EXTRA_ARGS=
Environment="KUBELET_EXTRA_ARGS=--node-ip=192.168.30.30"
```
This has to be followed with `systemctl daemon-reload && systemctl restart kubelet`.

And, in the end the Calico plugin:
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

## Network interfaces on master VM

```
vagrant@km:~$ ifconfig
(...)
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
(...)
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
(...)
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.30  netmask 255.255.255.0  broadcast 192.168.30.255
(...)
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
(...)
tunl0: flags=193<UP,RUNNING,NOARP>  mtu 1440
        inet 192.168.119.64  netmask 255.255.255.255
```

