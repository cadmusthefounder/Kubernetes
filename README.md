# Kubernetes

## Table of Contents

1. [Installing `kubeadm`](#installing-kubeadm)
2. [Useful General Commands](#useful-general-commands)
3. [Useful `kubectl` Commands](#useful-kubectl-commands)
4. [Useful `kubeadm` Commands](#useful-kubeadm-commands)

<a name="installing-kubeadm"/></a>
## Installing `kubeadm`


<a name="useful-general-commands"/></a>
## Useful General Commands

| Command | Description |
| --- | --- |
| `lsb_release -a` | Display the Ubuntu version. |
| `ifconfig -a` | Display all network interfaces with IPv4 and MAC address. |
| <code>ip route get 8.8.8.8 | awk '{print $NF; exit}'</code> | Display IP address that is currently used for Internet connections. |

<a name="useful-kubectl-commands"/></a>
## Useful `kubectl` Commands

| Command | Description |
| --- | --- |
| `kubectl version` | Display the Kubernetes version running on the client and server. |

<a name="useful-kubeadm-commands"/></a>
## Useful `kubeadm` Commands

| Command | Description |
| --- | --- |
