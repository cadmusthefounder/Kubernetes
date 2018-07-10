# Kubernetes

### Table of Contents

1. [Node Setup](#node-setup)
    1. [Check OS version](#check-os-version)
    2. [Check MAC address and product_uuid](#check-mac-and-uuid)
    3. [Configure firewall for Master node](#configure-master-firewall)
    4. [Configure firewall for Worker node](#configure-worker-firewall)
    5. [Installing kubeadm, kubelet and kubectl](#installing-kubeadm-kubelet-kubectl)
    6. [Configure cgroup driver used by kubelet on Master Node](#configure-cgroup)
2. [Useful Commands](#useful-commands)
    1. [General](#useful-general-commands)
    2. [ufw](#useful-ufw-commands)
    3. [kubeadm](#useful-kubeadm-commands)
    4. [kubelet](#useful-kubelet-commands)
    5. [kubectl](#useful-kubectl-commands)

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

<a name="installing-kubeadm-kubelet-kubectl"/></a>
### Installing kubeadm, kubelet and kubectl

Install these packages on all of your machines:

* kubeadm: the command to bootstrap the cluster.

* kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

* kubectl: the command line util to talk to your cluster.

``` bash
$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
apt-transport-https is already the newest version (1.2.27).
curl is already the newest version (7.47.0-1ubuntu2.8).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
OK
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
Hit:1 https://download.docker.com/linux/ubuntu xenial InRelease
Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [107 kB]                  
Fetched 363 kB in 2s (142 kB/s)       
Reading package lists... Done
$ sudo apt-get install -y kubelet kubeadm kubectl
Setting up cri-tools (1.11.0-00) ...
Setting up ebtables (2.0.10.4-3.4ubuntu2.16.04.2) ...
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
Setting up kubernetes-cni (0.6.0-00) ...
Setting up socat (1.7.3.1-1) ...
Setting up kubelet (1.11.0-00) ...
Setting up kubectl (1.11.0-00) ...
Setting up kubeadm (1.11.0-00) ...
Processing triggers for systemd (229-4ubuntu21.2) ...
Processing triggers for ureadahead (0.100.0-19) ...
```

<a name="configure-cgroup"/></a>
## Configure cgroup driver used by kubelet on Master Node

Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:

``` bash
docker info | grep -i cgroup
sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

If there are no `KUBELET_CGROUP_ARGS` in the **10-kubeadm.conf** file, add the following line:

``` bash
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```

Otherwise, update it with the following command:

``` bash
sudo sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Then restart kubelet:

``` bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

## Starting a pod

``` bash
sudo kubectl run simple-python-app \
     --image=afroisalreadyin/simple-python-app:v0.0.1 \
     --image-pull-policy=Never \
     --port=8080
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
| `ufw enable` | Enable ufw. |
| `ufw disable` | Disable ufw. Any rules created with UFW will no longer be active. |
| `ufw status verbose` | Check if ufw is running. Displays firewall rules if active. |
| `ufw allow 80`| Add rule to allow all traffic to port 80. |
| `ufw delete allow 80` | Remove rule from firewall. |

<a name="useful-kubeadm-commands"/></a>
### Useful `kubeadm` Commands

| Command | Description |
| --- | --- |
| `kubeadm token list` | List bootstrap tokens on the server. |
| `kubeadm token create` |  Create bootstrap tokens on the server. |
| `kubeadm token delete` | Delete bootstrap tokens on the server. |

<a name="useful-kubelet-commands"/></a>
### Useful `kubelet` Commands

| Command | Description |
| --- | --- |
| `journalctl -u kubelet` | See kubelet systemd logs. |

<a name="useful-kubectl-commands"/></a>
### Useful `kubectl` Commands

| Command | Description |
| --- | --- |
| `kubectl version` | Display the Kubernetes version running on the client and server. |
| `kubectl get nodes` | Display all nodes in cluster. |
| `kubectl drain <node name> --delete-local-data --force --ignore-daemonsets` | Safely evicts all pods from node. | 
| `kubectl delete node <node name>` | Remove node from cluster. | 


