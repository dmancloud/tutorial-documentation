# HA Kubernetes with RKE2 & Kube-VIP 
## Prerequsites
- 3 Virtual Machines (nodes) with Static IP Addresses
- Debian 11
- DNS configured for each of the nodes and the floating IP Address (VIP)
### ssh into the first master node
## Update Package Repository and Upgrade Packages
``` shell title="Run from shell prompt" linenums="1"
sudo apt-get update && sudo apt upgrade -y
sudo apt-get -y install gnupg2 ca-certificates curl apt-transport-https iptables
```

## Install kubectl
Additional Information - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt install ca-certificates curl apt-transport-https -y
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install kubectl -y
```

## Prepare configuration file for k8s-master01
``` shell title="Run from shell prompt" linenums="1"
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt" linenums="1"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below contents" linenums="1"
tls-san:
- k8s-master01
- k8s-master01.dman.cloud
- k8s-cluster.dman.cloud
- 192.168.3.83
disable: rke2-ingress-nginx
cni:
- calico
```

## Install RKE2 on k8s-master01 node
``` shell title="Export variables we will use to configure kube-vip" linenums="1"
export VIP=192.168.3.83
export TAG=v0.5.5
export INTERFACE=ens192
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
export PATH=/var/lib/rancher/rke2/bin:$PATH
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```
``` shell title="Let's create an alias to save us some time" linenums="1"
alias k=kubectl 
```
``` shell title="Install RKE on master node 1" linenums="1"  
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server
systemctl start rke2-server
```
### Copy Token
``` shell title="Run from shell prompt" linenums="1"
cat /var/lib/rancher/rke2/server/token
```
### Install kube-vip on k8s-master01 node
``` shell title="Run from shell prompt" linenums="1"
curl -s https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/rke2/server/manifests/kube-vip-rbac.yaml
crictl pull docker.io/plndr/kube-vip:$TAG
alias kube-vip="ctr --namespace k8s.io run --rm --net-host docker.io/plndr/kube-vip:$TAG vip /kube-vip"

kube-vip manifest daemonset \
--arp \
--interface $INTERFACE \
--address $VIP \
--controlplane \
--leaderElection \
--taint \
--services \
--inCluster | tee /var/lib/rancher/rke2/server/manifests/kube-vip.yaml
```
### Check to see if kube-vip pod is running
``` shell title="Run from shell prompt" linenums="1"
kubectl get pod -n kube-system | grep kube-vip
kubectl logs --tail 100 -n kube-system <pod_from_above> | | grep -i broad
```
### Check VIP Status
``` shell title="Run from shell prompt" linenums="1"
ping 192.168.3.83
```
## Prepare configuration file for k8s-master02 node
### ssh into the second master node
``` shell title="Run from shell prompt" linenums="1"
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt" linenums="1"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below values (change TOKEN)" linenums="1"
token: <PASTE TOKEN HERE>
server: https://k8s-cluster.dman.cloud:9345
tls-san:
- k8s-master02
- k8s-master02.dman.cloud
- k8s-cluster.dman.cloud
- 192.168.3.83
disable: rke2-ingress-nginx
cni:
- calico
```
``` shell title="Export variables we will use to configure kube-vip" linenums="1"
export VIP=192.168.3.83
export TAG=v0.5.5
export INTERFACE=ens192
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
export PATH=/var/lib/rancher/rke2/bin:$PATH
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
alias k=kubectl
```
``` shell title="Install RKE on master node 2" linenums="1"
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server
systemctl start rke2-server

```

## Prepare configuration file for k8s-master03 node
### ssh into the third master node
``` shell title="Run from shell prompt" linenums="1"
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt" linenums="1"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below values (change TOKEN)" linenums="1"
token: <PASTE TOKEN HERE>
server: https://k8s-cluster.dman.cloud:9345
tls-san:
- k8s-master03
- k8s-master03.dman.cloud
- k8s-cluster.dman.cloud
- 192.168.3.83
disable: rke2-ingress-nginx
cni:
- calico
```
``` shell title="Export variables we will use to configure kube-vip" linenums="1"
export VIP=192.168.3.83
export TAG=v0.5.5
export INTERFACE=ens192
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
export PATH=/var/lib/rancher/rke2/bin:$PATH
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
alias k=kubectl
```
``` shell title="Install RKE on master node 3" linenums="1"
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server
systemctl start rke2-server
```
## Check that kube-vip is running on all nodes
Go back to master node 1 where we installed kubectl
``` shell title="Run from shell prompt" linenums="1"
kubectl get pod -n kube-system | grep kube-vip
```
## Finally copy and edit the kubeconfig to talk to VIP
``` shell title="Run from shell prompt" linenums="1"
cp /etc/rancher/rke2/rke2.yaml .
vi rke2.yaml
```
Edit Server Address: https://127.0.0.1:6443 and replace with VIP
``` shell title="Run from shell prompt" linenums="1"
kubectl --kubeconfig ./rke2.yaml get nodes
```

You should now be able to test kube-vip is load balancing by shutting down one of the nodes and watching.