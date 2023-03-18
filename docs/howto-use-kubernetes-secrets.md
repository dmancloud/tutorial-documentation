# How to use Kubernetes Secrets
## Create Kubernetes secrets using kubectl and --from-literal
The easiest ways to create the Kubernetes secret is by using the kubectl command and --from-literal flag. For example to understand Kubernetes secret creation we need three things.

- secret-name - test-secret
- username - test-user
- password - testP@ssword

``` shell title="Run from shell prompt" linenums="1"
kubectl create secret generic test-secret --from-literal=username=test-user --from-literal=password=testP@ssword
```
### Verify the secret using the following command
``` shell title="Run from shell prompt" linenums="1"
kubectl get secret test-secret
```
### Describe The Secret
``` shell title="Run from shell prompt" linenums="1"
kubectl describe secret test-secret
```
### Base64 Encoded Kubernetes Secrets
``` shell title="Run from shell prompt" linenums="1"
echo -n ‘test-user’ | base64
```
## Using Kubernetes Secrets In A Deployment (mysql)
### Create a secret
``` yaml title="Create Secret" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: mysql-test-secret
type: kubernetes.io/basic-auth
stringData:
  password: test1234
```
### Create a deployment
``` yaml title="Create Deployment" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-test-secret
                  key: password
          ports:
            - containerPort: 3306
              name: mysql
```
