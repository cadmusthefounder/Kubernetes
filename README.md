# Kubernetes

### Table of Contents

1. [Node Setup](#node-setup)
    1. [Check OS version](#check-os-version)
    2. [Check MAC address and product_uuid](#check-mac-and-uuid)
    3. [Configure firewall for Master node](#configure-master-firewall)
    4. [Configure firewall for Worker node](#configure-worker-firewall)
2. [Useful Commands](#useful-commands)
    1. [General](#useful-general-commands)
    2. [**ufw**](#useful-ufw-commands)
    3. [**kubectl**](#useful-kubectl-commands)
    4. [**kubeadm**](#useful-kubeadm-commands)

<a name="node-setup"/></a>
## Node Setup

<a name="check-os-version"/></a>
### Check OS version

Check if Node is running Ubuntu 16.04 or later.

``` bash 
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:    Ubuntu 16.04.4 LTS
Release:        16.04
Codename:       xenial
```

<a name="check-mac-and-uuid"/></a>
### Check MAC address and product_uuid

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

<a name="configure-master-firewall"/></a>
### Configure firewall for Master node

Configure the firewall for Master node according to the following rules:

| Protocol | Direction | Port Range | Purpose |
| --- | --- | --- | --- |
| TCP | Inbound | 22 | SSH |
| TCP | Inbound | 6443 | Kubernetes API server |
| TCP | Inbound | 2379-2380 | etcd server client API |
| TCP | Inbound | 10250 | Kubelet API |
| TCP | Inbound | 10251 | kube-scheduler |
| TCP | Inbound | 10252 | kube-controller-manager |
| TCP | Inbound | 10255 | read-only Kubelet API |

Use UFW, or Uncomplicated Firewall, to help setup the above rules on the Master node.

``` bash
$ sudo ufw disable
Firewall stopped and disabled on system startup
$ sudo ufw status
Status: inactive
$ sudo ufw reset
Resetting all rules to installed defaults. This may disrupt existing ssh
connections. Proceed with operation (y|n)? y
$ sudo ufw allow 22
Rules updated
Rules updated (v6)
$ sudo ufw allow 6443
Rules updated
Rules updated (v6)
$ sudo ufw allow 2379:2380/tcp
Rules updated
Rules updated (v6)
$ sudo ufw allow 10250
Rules updated
Rules updated (v6)
$ sudo ufw allow 10251
Rules updated
Rules updated (v6)
$ sudo ufw allow 10252
Rules updated
Rules updated (v6)
$ sudo ufw allow 10255
Rules updated
Rules updated (v6)
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere                  
6443                       ALLOW IN    Anywhere                  
2379:2380/tcp              ALLOW IN    Anywhere                  
10250                      ALLOW IN    Anywhere                  
10251                      ALLOW IN    Anywhere                  
10252                      ALLOW IN    Anywhere                  
10255                      ALLOW IN    Anywhere                  
22 (v6)                    ALLOW IN    Anywhere (v6)             
6443 (v6)                  ALLOW IN    Anywhere (v6)             
2379:2380/tcp (v6)         ALLOW IN    Anywhere (v6)             
10250 (v6)                 ALLOW IN    Anywhere (v6)             
10251 (v6)                 ALLOW IN    Anywhere (v6)             
10252 (v6)                 ALLOW IN    Anywhere (v6)             
10255 (v6)                 ALLOW IN    Anywhere (v6)             
```

<a name="configure-worker-firewall"/></a>
### Configure firewall for Worker node

Configure the firewall for Worker node according to the following rules:

| Protocol | Direction | Port Range | Purpose |
| --- | --- | --- | --- |
| TCP | Inbound | 22 | SSH |
| TCP | Inbound | 10250 | Kubelet API |
| TCP | Inbound | 10255 | read-only Kubelet API |
| TCP | Inbound | 30000-32767 | NodePort Services |

Use UFW, or Uncomplicated Firewall, to help setup the above rules on the Worker node.

``` bash
$ sudo ufw disable
Firewall stopped and disabled on system startup
$ sudo ufw status
Status: inactive
$ sudo ufw reset
Resetting all rules to installed defaults. This may disrupt existing ssh
connections. Proceed with operation (y|n)? y
$ sudo ufw allow 22
Rules updated
Rules updated (v6)
$ sudo ufw allow 10250
Rules updated
Rules updated (v6)
$ sudo ufw allow 10255
Rules updated
Rules updated (v6)
$ sudo ufw allow 30000:32767/tcp
Rules updated
Rules updated (v6)
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere                  
10250                      ALLOW IN    Anywhere                  
10255                      ALLOW IN    Anywhere                  
30000:32767/tcp            ALLOW IN    Anywhere                  
22 (v6)                    ALLOW IN    Anywhere (v6)             
10250 (v6)                 ALLOW IN    Anywhere (v6)             
10255 (v6)                 ALLOW IN    Anywhere (v6)             
30000:32767/tcp (v6)       ALLOW IN    Anywhere (v6) 
```

<a name="useful-commands"/></a>
## Useful Commands

<a name="useful-general-commands"/></a>
### Useful General Commands

| Command | Description |
| --- | --- |
| `lsb_release -a` | Display the Ubuntu version. |
| `ifconfig -a` | Display all network interfaces with IPv4 and MAC address. |
| `ip route get 8.8.8.8 \| awk '{print $NF; exit}'` | Display IP address that is currently used for Internet connections. |

<a name="useful-ufw-commands"/></a>
### Useful `ufw` Commands
| Command | Description |
| --- | --- |
| `sudo ufw enable` | Enable ufw. |
| `sudo ufw disable` | Disable ufw. Any rules created with UFW will no longer be active. |
| `sudo ufw status verbose` | Check if ufw is running. Displays firewall rules if active. |
| `sudo ufw allow 80`| Add rule to allow all traffic to port 80. |
| `sudo ufw delete allow 80` | Remove rule from firewall. |


<a name="useful-kubectl-commands"/></a>
### Useful `kubectl` Commands

| Command | Description |
| --- | --- |
| `kubectl version` | Display the Kubernetes version running on the client and server. |

<a name="useful-kubeadm-commands"/></a>
### Useful `kubeadm` Commands

| Command | Description |
| --- | --- |
