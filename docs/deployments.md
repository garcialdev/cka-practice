# Kubernetes Deployments Guide

### Instructions on how to create a minimal deployment

```bash
# This command will show a minimal output of a deployment for the httpd image with 10 replicas
kubectl create deploy test--image=httpd --replicas=10 --dry-run=client -o yaml

# Add > to redirect the output to a file named deployment.yaml
kubectl create deploy test --image=httpd --replicas=10 --dry-run=client -o yaml > deployment.yaml

# test is the name given to the deployment
```

This will provide you with an output like the following:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: httpd
        name: httpd
        resources: {}
status: {}
```

I've removed resources and status since we do not need this. 
Below is a simplified example where I have added the type of update strategy we want to use and configured the maxUnavailable and maxSurge:

```yaml
apiVersion: apps/v1 # Specifies the API version
kind: Deployment # Specifies that this is a Deployment resource
metadata:
  labels:
    app: test # Label to identify the application
  name: test # Name of the deployment
spec:
  replicas: 10 # Number of desired pod replicas
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - image: httpd:trixie # you can update the tag as needed, kubernetes will pull the latest version if not specified.
          name: httpd
  strategy:
    type: RollingUpdate # Default is RollingUpdate, can be omitted
    rollingUpdate:
      maxUnavailable: 1 # Default is 25%, can be omitted. This means at most 1 pod can be unavailable during the update.
      maxSurge: 1 # Default is 25%, can be omitted. This means at most 1 extra pod can be created during the update.
```