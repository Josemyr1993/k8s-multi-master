# k8s-multi-master
Configuring multi-master control-plane and haproxy

![image](https://user-images.githubusercontent.com/86851766/208912472-5a6697a4-85b1-4202-ab81-296840bb4072.png)

# Definition

HAProxy is a free, very fast and reliable reverse-proxy offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for very high traffic web sites and powers a significant portion of the world's most visited ones. Over the years it has become the de-facto standard opensource load balancer, is now shipped with most mainstream Linux distributions, and is often deployed by default in cloud platforms. Since it does not advertise itself, we only know it's used when the admins report it.

# Requisites

3 - Ubuntu machines,  distro: ubuntu  22.04

Virtualbox

Host machine has atleast 8 cores

Host machine has atleast 8G memory

# From loadbalancer machine (loadbalancer.example.net) as root
```
apt update && apt install -y haproxy
```
Configure haproxy
Append the below lines to /etc/haproxy/haproxy.cfg
```
frontend kubernetes-frontend
    bind 192.168.32.144:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server kmaster1 192.168.32.145:6443 check fall 3 rise 2
    server kmaster2 192.168.32.146:6443 check fall 3 rise 2
```

Restart haproxy service
```
systemctl restart haproxy
```

# On all kubernetes nodes (kmaster1, kmaster2, kworker1)

Disable Firewall

```
ufw disable
```

Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
Update sysctl settings for Kubernetes networking

```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
Install docker engine

```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update && apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}

```
Kubernetes Setup
Add Apt repository

```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}

Install Kubernetes components

```
apt update && apt install -y kubeadm=1.19.2-00 kubelet=1.19.2-00 kubectl=1.19.2-00

#  On any one of the Kubernetes master node (Eg: kmaster1)

Initialize Kubernetes Cluster

```
kubeadm init --control-plane-endpoint="ip_loadbalancer:6443" --upload-certs --apiserver-advertise-address=ip_do_kmaster1 --pod-network-cidr=192.168.0.0/16

Copy the commands to join other master nodes and worker nodes.

Deploy Calico network from kmaster1

```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```
# Join other nodes to the cluster (kmaster2 & kworker1)
Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

IMPORTANT: You also need to pass --apiserver-advertise-address to the join command when you join the other master node.

EXEMPLE:  
```
ubeadm join 192.168.32.144:6443 --token nmhi16.2k8s946cngngpel6     --discovery-token-ca-cert-hash sha256:c180ae6ab9ae309dcd8667851c56481a155dd0e4d3d1d0d750b5f399497d25cf     --control-plane --certificate-key 887512530c6ec111e5173a1eec9015f24303d505807e4face496ce29f40f1a34 --apiserver-advertise-address 192.168.32.146
```


