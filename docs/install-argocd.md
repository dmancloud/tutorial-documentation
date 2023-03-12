# ArgoCD Installation
Argo CD is a declarative continuous delivery tool for Kubernetes applications. It uses the GitOps style to create and manage Kubernetes clusters. When any changes are made to the application configuration in Git, Argo CD will compare it with the configurations of the running application and notify users to bring the desired and live state into sync.

## Prerequsites 
- Virtual Machine running Ubuntu 22.04 or newer
### Update Package Repository and Upgrade Packages
``` shell title="Run from shell prompt" linenums="1"
sudo apt update
sudo apt upgrade
```
## Create Kubernetes Cluster
``` shell title="Run from shell prompt" linenums="1"
sudo bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --disable traefik
exit 
mkdir .kube
sudo cp /etc/rancher/k3s/k3s.yaml ./config
sudo chown dmistry:dmistry config
chmod 400 config
export KUBECONFIG=~/.kube/config
```

### Install ArgoCD
``` shell title="Run from shell prompt" linenums="1"
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### Change Service to NodePort
Edit the service can change the service type from `ClusterIP` to `NodePort`
``` shell title="Run from shell prompt" linenums="1"
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}' 
```
### Fetch Password
``` shell title="Run from shell prompt" linenums="1"
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
# Deploy Demo Application
You can use the below repository to deploy a demo nginx application
``` shell title="This repository has a sample application" linenums="1"
https://github.com/dmancloud/argocd-tutorial
```
### Scale Replicaset 
``` shell title="Run from shell prompt" linenums="1"
kubectl scale --replicas=3 deployment nginx -n default
```

## Clean Up
``` shell title="Run from shell prompt" linenums="1"
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```