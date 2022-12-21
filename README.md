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

# From loadbalancer machine (loadbalancer.example.net)
```
apt update && apt install -y haproxy
```
