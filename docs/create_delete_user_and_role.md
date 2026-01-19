# 👤 How to create a new user

```bash
openssl genrsa -out user.key 2048
```

```bash
openssl req -new -key user.key -out user.csr -subj "/CN=user/O=readers"
```

```bash
sudo openssl x509 -req \
  -in user.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out user.crt \
  -days 365
```

```
kubectl config set-credentials user \
  --client-certificate=user.crt \
  --client-key=user.key
```

```bash
kubectl config set-context user@k8s \
  --cluster=kubernetes \
  --namespace=default \
  --user=user
```

# Create Role with permissions to read pods

Create a file named role.yaml with the following content:

```bash
vim role.yaml
```


```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Apply the Role:

```bash
kubectl apply -f role.yaml
```

# Create a RoleBinding (Declarative YAML)

Lastly, bind the Role to the user by creating a RoleBinding. Create a file named rolebinding.yaml with the following content:

```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
  - kind: User
    name: user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

Apply the RoleBinding:

```bash
kubectl apply -f rolebinding.yaml
```

# Test the RBAC Permissions
Now you can test the permissions of the user user:

```bash
kubectl config use-context user@k8s
```

This should work

```bash
kubectl get pods
```

This should fail

```bash
kubectl delete pod <pod-name>
``` 

Test using can-i

```bash
kubectl auth can-i list pods
kubectl auth can-i delete pods
```

# Remove Everything (Declarative Deletion)

Delete rbac resources created:

```bash

kubectl delete -f rolebinding.yaml
kubectl delete -f role.yaml
```

Remove kubeconfig entries:
```bash
kubectl config delete-context user@k8s
kubectl config delete-user user
```

Delete certificate + key from disk:
```bash
rm user.key user.crt user.csr
```