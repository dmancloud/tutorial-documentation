# TLS Certifcates on Kubernetes with Traefik and Cloudflare

In this tutorial we will deploy Traefik as our ingress controller and use Cloudflare and Let's Encrypt to secure our applications running in our kubernetes clusters

## Prerequsites 
- Kubernetes Cluster
- Helm installed

If you have not already done so make sure you have exported your kubeconig so you can access the cluster 

``` shell title="export your kubeconfig" linenums="1"
export KUBECONFIG=/home/dmistry/.kube/k8s-cluster.dev.dman.cloud.yaml
```

## Clone Repository 
``` shell title="export your kubeconfig" linenums="1"
git clone git@github.com:dmancloud/traefik-cert-manager.git
```

## Install Traefik Ingress Controller
``` shell title="export your kubeconfig" linenums="1"
helm repo add traefik https://helm.traefik.io/traefik
```

Update and make any changes need to the values file. If yo want to fetch the complete values and make additional adjustments you can do so by running the following command
``` shell title="export your kubeconfig" linenums="1"
helm show values traefik/traefik > /tmp/values.yaml
```
Install Traefik
``` shell title="export your kubeconfig" linenums="1"
helm install --namespace=traefik traefik traefik/traefik --values=values.yaml
```
Add default header values needed by most applications
``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f default-headers.yaml
```

## Install Cert-Manager
``` shell title="export your kubeconfig" linenums="1"
helm repo add jetstack https://charts.jetstack.io
```
``` shell title="export your kubeconfig" linenums="1"
helm upgrade --install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0 \
  --set installCRDs=true \
  --values=values.yaml \
  --create-namespace
```

Next we need to create an API token on CloudFlare so we can create a secret for Lets Encrypt to use. Edit the `secret-cf-token.yaml` and replace the `cloudflare-token:  ` with your token.

When creating your token on Cloudflare you need to make sure you grant `edit` access to the token

``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f secret-cf-token.yaml
```

Create ClusterIssuer you should start with a Staging Certificate before moving to Production certifcates to avoid any rate limiting in case you make a mistake.

Edit the `letsencrypt-staging.yaml` and `letsencrypt-production.yaml` files and adjust to match your setup
``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f letsencrypt-staging.yaml
kubectl apply -f letsencrypt-production.yaml
```
## Create Certificate for you Service (nginx)

Next we will want to create a certificate for your service, you will need to make sure that you have created a DNS entry for your FQDN.
``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f nginx-dev.dman-cloud.yaml
```

## Deploy your application and configure IngressRoute
In this example we will deploy nginx and create a simple IngressRoute

``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Next we will deploy an IngressRoute be sure to edit the `ingress.yaml` and make any adjustments like your domain name, tls certifcate etc

``` shell title="export your kubeconfig" linenums="1"
kubectl apply -f ingress.yaml
```