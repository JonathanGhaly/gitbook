# 🙆♂ Initialize Kubeadm On Master Node To Setup Control Plane

Set the following environment variables. Replace `10.53.140.249` with the IP of your master node.

```bash
IPADDR="10.53.140.249"
NODENAME=$(hostname -s)
POD_CIDR="10.53.140.0/24"
```

Initialize the master node control plane configurations using the kubeadm command.

```bash
sudo kubeadm init --apiserver-advertise-address=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap
```

`--ignore-preflight-errors Swap` is actually not required as we disabled the swap initially

Use the following commands from the output to create the `kubeconfig` in master so that you can use `kubectl` to interact with cluster API.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Now, verify the kubeconfig by executing the following kubectl command to list all the pods in the `kube-system` namespace.

```bash
kubectl get po -n kube-system
```
