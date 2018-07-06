## Kubernetes

### Table of Contents

1. [Node Setup](#node-setup)
2. [Useful General Commands](#useful-general-commands)
3. [Useful **kubectl** Commands](#useful-kubectl-commands)
4. [Useful **kubeadm** Commands](#useful-kubeadm-commands)

<a name="node-setup"/></a>
### Node Setup

Check if Node is running Ubuntu 16.04 or later.

``` bash 
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.4 LTS
Release:		16.04
Codename:		xenial
```

Check for unique MAC address and product__uuid for every node.

``` bash
$ ifconfig -a
enp3s0    Link encap:Ethernet  HWaddr d0:50:99:1a:77:24  
          inet addr:10.10.10.14  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: fe80::d250:99ff:fe1a:7724/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1704065 errors:0 dropped:0 overruns:0 frame:0
          TX packets:378189 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:2261685790 (2.2 GB)  TX bytes:28254676 (28.2 MB)
          Memory:f7200000-f727ffff 
$ sudo cat /sys/class/dmi/id/product_uuid
1100XXXX-XXXX-XXX-XXXX-XXXXXXXX009
```
<a name="useful-general-commands"/></a>
### Useful General Commands

| Command | Description |
| --- | --- |
| `lsb_release -a` | Display the Ubuntu version. |
| `ifconfig -a` | Display all network interfaces with IPv4 and MAC address. |
| `ip route get 8.8.8.8 \| awk '{print $NF; exit}'` | Display IP address that is currently used for Internet connections. |

<a name="useful-kubectl-commands"/></a>
### Useful `kubectl` Commands

| Command | Description |
| --- | --- |
| `kubectl version` | Display the Kubernetes version running on the client and server. |

<a name="useful-kubeadm-commands"/></a>
### Useful `kubeadm` Commands

| Command | Description |
| --- | --- |
