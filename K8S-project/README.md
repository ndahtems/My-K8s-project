How to Provision VM for a Kubernetes Cluster with 1Master $ 2Worker Nodes:
==========================================================================
## 1. Prerequisite: VirtualBox and Vagrant
Select the appropriate OS
```sh
https://download.virtualbox.org/virtualbox/7.0.10/VirtualBox-7.0.10-158379-Win.exe
```
## Follow the link to install Vagrant:
```sh
https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-install
```
You can use an already configured vagrant from;
```sh
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
```
cd dir
- vagrant status
- vagrant up
- vagrant status
- ssh into all the nodes
- vagrant ssh kubemaster

## Install kubeadm Doc:
```ss
  https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```
Installing a container runtime:
Select contairnerd

Forwarding IPv4 and letting iptables see bridged traffic:

#Run this on all nodes
```ssh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
#sysctl params required by setup, params persist across reboots
```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that the br_netfilter, overlay modules are loaded by running the following commands:
```sh
lsmod | grep br_netfilter
lsmod | grep overlay
```
Verify that all system variables are set to 1:
```sh
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```
## Install contairnerd

https://github.com/containerd/containerd/blob/main/docs/getting-started.md
```sh
https://docs.docker.com/engine/install/ubuntu/
```
Follow the steps to install using the Apt repository
Run instruction 1 on all nodes

run the command on all nodes:
```sh
  sudo apt install containerd.io
  systemctl status containerd
```
check you init system:
```sh
  ps -p 1
```
## Configuring the systemd cgroup driver in all nodes:
#To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
```sh
sudo vi /etc/containerd/config.toml
```
delete all file content and paste the following:
```sh
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true  
```
```sh
sudo systemctl restart containerd 
```
## Installing kubeadm, kubelet and kubectl:
- kubeadm: the command to bootstrap the cluster.
- kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
- kubectl: the command line util to talk to your cluster.

Select Debian based Distributions;
Follow the steps and run on all nodes
```sh
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Create a Cluster:
```sh
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
```
## 1. To initialize the control-plane node run (master-only):

  to get the ip address, run ip add
```sh  
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.2
```
#Copied the displayed commands and run on the control-plane
```sh
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config       
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
#do not delete the output

## deploy a pod network to the cluster:
```sh
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```
Select weave network  
Run this command on the controlplane:
```sh
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
kubectl edit ds weave-net -n kube-system
Under containers, go to the weave container and add a new env 
```sh
- name: IPALLOC_RANGE
  value: 10.244.0.0/16
```
## Join the worker nodes to the cluster:
Run this on all worker nodes
```sh
 sudo kubeadm join 192.168.56.2:6443 --token 18314j.ny40ysss2jnjgz0u \ 
        --discovery-token-ca-cert-hash sha256:5395b66c9b96156ecda4f35caa8f08e7447e6432f77428fbe0202893e502e61f
```
kubectl get nodes
kubectl get pods -A 
kubectl get pods -n kube-system
