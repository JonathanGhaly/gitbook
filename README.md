---
description: On this page the steps to install k8s
---

# Installation

## 1- Container Runtimes Installation

### Configuration and Prerequisites:

#### Forwarding IPv4 and letting iptables see bridged traffic <a href="#forwarding-ipv4-and-letting-iptables-see-bridged-traffic" id="forwarding-ipv4-and-letting-iptables-see-bridged-traffic"></a>

Execute the below-mentioned instructions:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that the `br_netfilter`, `overlay` modules are loaded by running the following commands:

<pre class="language-bash"><code class="lang-bash"><strong>lsmod | grep br_netfilter
</strong>lsmod | grep overlay
</code></pre>

Verify that the `net.bridge.bridge-nf-call-iptables`, `net.bridge.bridge-nf-call-ip6tables`, and `net.ipv4.ip_forward` system variables are set to `1` in your `sysctl` config by running the following command:

```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forwardcgroup drivers
```

### cgroup drivers <a href="#cgroup-drivers" id="cgroup-drivers"></a>

There is two types of cgroup drivers

* cgroupfs
* systemd

If you use a system that contains `systemd` _as_ cgroup driver as init, it's preferred not to use `cgroupfs`

#### cgroupfs driver <a href="#cgroupfs-cgroup-driver" id="cgroupfs-cgroup-driver"></a>

The `cgroupfs` driver is the [default cgroup driver in the kubelet](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1). When the `cgroupfs` driver is used, the kubelsystemdet and the container runtime directly interface with the cgroup filesystem to configure cgroups.

#### systemd cgroup driver <a href="#systemd-cgroup-driver" id="systemd-cgroup-driver"></a>

To set `systemd` as the cgroup driver, edit the [`KubeletConfiguration`](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/) option of `cgroupDriver` and set it to `systemd`. For example:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd
```

If you configure `systemd` as the cgroup driver for the kubelet, you must also configure `systemd` as the cgroup driver for the container runtime. Refer to the documentation for your container runtime for instructions.

### Container runtimes[ ](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#container-runtimes)

#### Containerd

#### &#x20; 
