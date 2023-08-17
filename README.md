# 🛠 Node Installation

## Prerequisites:

```bash
dnf install -y iproute-tc
```

## Docker Instillation:

update the package database:

```bash
dnf check-update
```

Add the official Docker repository:

```bash
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

While there is no Rocky Linux specific repository from Docker, Rocky Linux is based upon CentOS and can use the same repository. With the repository added, install Docker, which is composed of three packages:

```bash
dnf install docker-ce docker-ce-cli containerd.io -y
```

Start Docker:

```bash
systemctl start docker
```

```bash
systemctl status docker
```

Change between enforcing and permissive mode

```bash
setenforce 0
```

Disable SELinux

```bash
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
```

```bash
sestatus
```

Result:

```
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

## K8S Install

Configure network k8s.config

```bash
nano /etc/sysctl.d/k8s.conf
```

add the following to the file:

```
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

Apply the changes in sysctl

```bash
sysctl --system
```

Disable Swap of vmemory:

```bash
swapoff -a
```

Make changes across reboots:

```bash
sed -e '/swap/s/^/#/g' -i /etc/fstab
```

Add k8s repo to system repos:

```bash
nano /etc/yum.repos.d/k8s.repo
```

k8s.repo:

```
[kubernetes]                                                                                         
name=Kubernetes                                                                                      
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64                            
enabled=1                                                                                            
gpgcheck=1                                                                                           
repo_gpgcheck=1                                                                                      
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

Refresh the local repository cache:

```bash
dnf makecache -y
```

Install the packages with following command

```bash
dnf install -y {kubelet,kubeadm,kubectl} --disableexcludes=kubernetes
```

Enable kubelet

```bash
systemctl enable kubelet.service
```

### Install CRI-O Runtime On All The Nodes <a href="#install-cri-o-runtime-on-all-the-nodes" id="install-cri-o-runtime-on-all-the-nodes"></a>

Enable **cri-o** repositories for version 1.26

```bash
VERSION=1.26
```

Add the gpg keys.

```bash
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/CentOS_8/devel:kubic:libcontainers:stable.repo
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:${VERSION}.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:${VERSION}/CentOS_8/devel:kubic:libcontainers:stable:cri-o:${VERSION}.repo
```

List repositories available on the system

```bash
sudo dnf -y repolist
repo id                                                                               repo name
appstream                                                                             AlmaLinux 8 - AppStream
baseos                                                                                AlmaLinux 8 - BaseOS
devel_kubic_libcontainers_stable                                                      Stable Releases of Upstream github.com/containers packages (CentOS_8)
devel_kubic_libcontainers_stable_cri-o_1.26                                           devel:kubic:libcontainers:stable:cri-o:1.26 (CentOS_8)
extras           
```

With the repository configured, installCRI-O Container runtime on Rocky Linux 8|AlmaLinux 8

```bash
sudo dnf install cri-o cri-tools --allowerasing -y
```

Reload the systemd configurations and enable cri-o.

```bash
systemctl daemon-reload
systemctl enable crio --now
```

