# Prepare all nodes
{
apt-get update
apt-get install -y apt-transport-https ca-certificates curl

cat <<EOF > /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system

apt-get install -y containerd

mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

systemctl restart containerd

KUBE_LATEST=$(curl -L -s https://dl.k8s.io/release/stable.txt | awk 'BEGIN { FS="." } { printf "%s.%s", $1, $2 }')

mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/ /" > /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

crictl config \
    --set runtime-endpoint=unix:///run/containerd/containerd.sock \
    --set image-endpoint=unix:///run/containerd/containerd.sock

cat <<EOF > /etc/default/kubelet
KUBELET_EXTRA_ARGS='--node-ip $(ip -4 addr show enp0s8 | grep "inet" | head -1 |awk '{print $2}' | cut -d/ -f1)'
EOF
}

# Install Controlplane

{
POD_CIDR=10.244.0.0/16
SERVICE_CIDR=10.96.0.0/16
PRIMARY_IP=192.168.56.11/24

kubeadm init --pod-network-cidr $POD_CIDR --service-cidr $SERVICE_CIDR --apiserver-advertise-address $PRIMARY_IP

kubectl --kubeconfig /etc/kubernetes/admin.conf \
    apply -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s-1.11.yaml"

}

# Join the nodes

Join Workers

If you did not note down the join command on the controlplane node after running `kubeadm`, you can recover it by running the following on `controlplane`

```bash
kubeadm token create --print-join-command
```

[//]: # (host:node01-node02)
[//]: # (comment:Run kubeadm join)

On each of `node01` and `node02` do the following

1.  Become root (if you are not already)

    ```
    sudo -i
    ```

1.  Join the node

    > Paste the `kubeadm join` command output by `kubeadm init` on the control plane