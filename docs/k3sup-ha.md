# Lightweight HA Kubernetes with k3s & kube-vip
## Perfect kuberntes cluster for homelabs

- k3s is lightweight kubernetes, easy to install, 50% memory, single binary of less than 100 MB.  
- kube-vip provides kubernetes clusters with a virtual IP for the control plane  
- k3sup is a light-weight utility to get from zero to KUBECONFIG with k3s on any local or remote VM
## Prerequsites

!!! info "System Requirements"
    Five (5) linux virtual machines with statically configured IPs. It is recommended that the virtual machines have an **A Record** pointing to the IP address of the host.  
      
    A floating IP Address for the Virtual IP to access the cluster. It is recommended that to have an **A Record** pointing to the floating IP Address.

    You will also need a linux host that you will deploy the server and agent nodes from.

## Install k3sup
First you will need to install k3sup which is what we will use to deploy the server and agent nodes from.

``` shell title="Install k3sup"
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/

k3sup --help
```
k3sup uses ssh to connect to the server and agent nodes so we need to copy our public key.

``` shell title="copy ssh key to all nodes"
ssh-copy-id dmistry@192.168.1.21  
ssh-copy-id dmistry@192.168.1.22  
ssh-copy-id dmistry@192.168.1.23  
ssh-copy-id dmistry@192.168.1.24  
ssh-copy-id dmistry@192.168.1.25  
```

Next we deploy a k3s sever node to the first node

``` shell linenums="1"
k3sup install --ip 192.168.1.21 \
--user dmistry \
--sudo \
--tls-san 192.168.1.20 \
 --cluster --local-path ~/.kube/k8s-cluster.dev.dman.cloud.yaml \
 --context k8s-cluster-ha \
 --k3s-extra-args "--disable traefik --node-ip=192.168.1.21"
```
``` shell
export KUBECONFIG=~/.kube/k8s-cluster.dev.dman.cloud.yaml
```

``` shell
kubectl apply -f https://kube-vip.io/manifests/rbac.yaml 
```
SSH in to the first server node and install kube-vip
``` shell
ssh 192.168.1.21
```
``` shell
sudo -i
```
``` shell
ctr image pull docker.io/plndr/kube-vip:latest
```
``` shell
alias kube-vip="ctr run --rm --net-host docker.io/plndr/kube-vip:latest vip /kube-vip"
```
``` shell linenums="1"
kube-vip manifest daemonset \
--arp \
--interface ens192 \
--address 192.168.1.20 \
--controlplane \
--leaderElection \
--taint \
--inCluster | tee /var/lib/rancher/k3s/server/manifests/kube-vip.yaml
```
Logout of first server node and join serves node 2 and server node 3

``` shell title="Server Node 2"
k3sup join --ip 192.168.1.22 --user dmistry --sudo --k3s-channel stable --server --server-ip 192.168.1.20 --server-user dmistry --sudo --k3s-extra-args "--disable traefik  --node-ip=192.168.1.22"
```
``` shell title="Server Node 3"
k3sup join --ip 192.168.1.23 --user dmistry --sudo --k3s-channel stable --server --server-ip 192.168.1.20 --server-user dmistry --sudo --k3s-extra-args "--disable traefik  --node-ip=192.168.1.23"
```
Next we configure the agent nodes
``` shell title="Agent Node 2"
k3sup join --user dmistry --sudo --server-ip 192.168.1.20 --ip 192.168.1.24 --k3s-channel stable -- --k3s-extra-args "--disable traefik" --print-command
```
``` shell title="Agent Node 2"
k3sup join --user dmistry --sudo --server-ip 192.168.1.20 --ip 192.168.1.25 --k3s-channel stable -- --k3s-extra-args "--disable traefik" --print-command
```

You can now download the kubeconfig from server node 1 and update the IP adderss to match the load balancer IP (192.168.1.20)





