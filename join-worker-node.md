# 🔗 Join Worker Node

## prerequisites:

We have set up cri-o, kubelet, and kubeadm utilities on each node

## Joining:

If you missed copying the join command, execute the following command in the master node to recreate the token with the join command.

```bash
kubeadm token create --print-join-command
```

Here is what the command looks like. Use `sudo` if you running as a normal user. This command performs the [TLS bootstrapping](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) for the nodes.

```bash
sudo kubeadm join 10.53.140.249:6443 --token j4eice.33vgvgyf5cxw4u8i \
    --discovery-token-ca-cert-hash sha256:37f94469b58bcc8f26a4aa44441fb17196a585b37288f85e22475b00c36f1c61
```

Now execute the **kubectl command from the master node** to check if the node is added to the master.

```bash
kubectl get nodes
```

Example output,

```bash
root@master-node:/home/vagrant# kubectl get nodes
NAME            STATUS   ROLES           AGE     VERSION
master-node     Ready    control-plane   14m     v1.24.6
worker-node01   Ready    <none>          2m13s   v1.24.6
worker-node02   Ready    <none>          2m5s    v1.24.6
```

\
In the above command, the ROLE is `<none>` for the worker nodes. You can add a label to the worker node using the following command. Replace `worker-node01` with the hostname of the worker node you want to label.

```
kubectl label node worker-node01  node-role.kubernetes.io/worker=worker
```
