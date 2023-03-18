# Argo CD Git Webhook Configuration
By default, Argo CD polls the Git repositories every 3 minutes to detect the changes made on the repo. If you want to remove the delay, you can configure a webhook event to send a notification to the API server.
## Create a Webhook in GitHub
Login to your Github repository and navigate to settings > webhooks and click ```add webhook```

The payload URL is your ArgoCD Server + /api/webhook

For example.

```https://argocd.example.com/api/webhook```

If you wish to use a shared secret, input a value in the secret, remember this as we will create a secret in the argocd installation below.

Content type needs to be set to ```application/json```

Note: If ArgoCD is publicly accessible, then configuring a webhook secret is highly recommended.

## Create a Webhook Secret in ArgoCD
Edit the Argo CD kubernetes secret
``` shell title="Run from shell prompt" linenums="1"
kubectl edit secret argocd-secret -n argocd
```
TIP: for ease of entering secrets, kubernetes supports inputting secrets in the stringData field, which saves you the trouble of base64 encoding the values and copying it to the data field. Simply copy the shared webhook secret created in step 1, to the stringData field
``` yaml title="Edit secrets and apply" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: argocd-secret
  namespace: argocd
type: Opaque
data:
...

stringData:
  webhook.github.secret: SecretCreatedInGithub
```

