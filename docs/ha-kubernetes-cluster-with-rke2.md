# HA Kubernetes with RKE2 & Kube-VIP
kube-vip provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type LoadBalancer without relying on any external hardware or software.

## Prerequsites

!!! tip "System Requirements"
    Three (3) linux virtual machines with statically configured IPs. It is recommended that the virtual machines have an **A Record** pointing to the IP address of the host.  
      
    A floating IP Address for the Virtual IP to access the cluster. It is recommended that to have an **A Record** pointing to the floating IP Address.


## Configure the first master node
``` shell title="Become root"
sudo -i
```
### Update Package Repository and Upgrade Packages
``` shell title="Run from shell prompt"
apt-get update && sudo apt upgrade -y
```
```sh title="Run from shell prompt"
apt-get -y install gnupg2 ca-certificates \
curl apt-transport-https iptables
```

### Install kubectl (optional if already installed)
Additional Information - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
``` shell title="Run from shell prompt" linenums="1"
apt update
apt install ca-certificates curl apt-transport-https -y
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update
apt install kubectl -y
```

### Prepare configuration file for k8s-master01

``` shell title="Run from shell prompt"
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below contents" linenums="1"
tls-san:
- k8s-master01
- k8s-master01.dev.dman.cloud
- k8s-cluster.dev.dman.cloud
- 192.168.1.20
disable: rke2-ingress-nginx
cni:
- calico
```


### Install RKE2 on k8s-master01 node
``` shell title="Export variables we will use to configure kube-vip" linenums="1"
export VIP=192.168.1.20
export TAG=v0.5.11
export INTERFACE=ens192
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
export PATH=/var/lib/rancher/rke2/bin:$PATH
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```
``` shell title="Let's create an alias to save us some time"
alias k=kubectl 
```

``` shell title="Install RKE2 on master node 1" 
curl -sfL https://get.rke2.io | sh -
```
```sh
systemctl enable rke2-server
```
``` shell
systemctl start rke2-server
```

### Copy Token and Save
``` shell title="Run from shell prompt"
cat /var/lib/rancher/rke2/server/token
```
### Install kube-vip on k8s-master01 node
``` shell title="Configure roles for kube-vip"
curl -s https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/rke2/server/manifests/kube-vip-rbac.yaml
```
``` shell title="Pull latest kube-vip"
crictl pull docker.io/plndr/kube-vip:$TAG
```
``` shell title="Create an alias for kube-vip to save time"
alias kube-vip="ctr --namespace k8s.io run --rm --net-host docker.io/plndr/kube-vip:$TAG vip /kube-vip"
```
``` shell title="Create a daemonset manifest to run kube-vip"
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
``` shell title="Find the running kube-vip pods" 
kubectl get pod -n kube-system | grep kube-vip
```
``` shell title="Find the node elected as leader" 
kubectl logs --tail 100 -n kube-system <pod_from_above> | grep -i leader
```
### Verify the floating IP Status
``` shell title="Run from shell prompt"
ping 192.168.1.20
```

## Prepare configuration file for k8s-master02 node
### Login into the second master node
``` shell title="Become root"
sudo -i
```
``` shell title="Run from shell prompt"
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below values remember to use the token copied above" linenums="1"
token: <PASTE TOKEN HERE>
server: https://k8s-cluster.dev.dman.cloud:9345
tls-san:
- k8s-master02
- k8s-master02.dev.dman.cloud
- k8s-cluster.dev.dman.cloud
- 192.168.1.20
disable: rke2-ingress-nginx
cni:
- calico
```

``` shell title="Download RKE2"
curl -sfL https://get.rke2.io | sh -
```
``` shell title="Enable RKE2"
systemctl enable rke2-server
```
``` shell title="Start RKE2"
systemctl start rke2-server
```

## Prepare configuration file for k8s-master03 node
### Login into the third master node
``` shell title="Become root"
sudo -i
```

``` shell title="Run from shell prompt" 
mkdir -p /etc/rancher/rke2
```
``` shell title="Run from shell prompt"
vi /etc/rancher/rke2/config.yaml
```
``` shell title="Paste the below values remember to use the token copied above" linenums="1"
token: <PASTE TOKEN HERE>
server: https://k8s-cluster.dev.dman.cloud:9345
tls-san:
- k8s-master03
- k8s-master03.dev.dman.cloud
- k8s-cluster.dev.dman.cloud
- 192.168.1.20
disable: rke2-ingress-nginx
cni:
- calico
```

``` shell title="Download RKE2"
curl -sfL https://get.rke2.io | sh -
```
``` shell title="Enable RKE2"
systemctl enable rke2-server
```
``` shell title="Start RKE2"
systemctl start rke2-server
```

## Check that kube-vip is running on all nodes
Go back to master node 1 where we installed kubectl
``` shell title="Run from shell prompt"
kubectl get pod -n kube-system | grep kube-vip
```
## Finally copy and edit the kubeconfig to talk to VIP
``` shell title="Run from shell prompt"
cp /etc/rancher/rke2/rke2.yaml .
```
``` shell title="Run from shell prompt"
vi rke2.yaml
```
Edit Server Address: https://127.0.0.1:6443 and replace with VIP
``` shell title="Run from shell prompt"
kubectl --kubeconfig ./rke2.yaml get nodes
```

You should now be able to test kube-vip is load balancing by shutting down one of the nodes and watching.