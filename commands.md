# 📘 Kubernetes Command Cheat Sheet

A personal collection of useful `kubectl` and Kubernetes-related commands for quick reference and study.
Feel free to fork or clone this file to your GitHub repo!

---

## 🚀 Pod Management

```bash
kubectl get pods                                # List all pods in the current namespace
kubectl get pods -A                             # List all pods across all namespaces
kubectl get pods --field-selector=status.phase=Running  # List only running pods
kubectl get all                                 # List all resources in the current namespace
kubectl describe pod <pod-name>                 # Show detailed information about a specific pod
kubectl logs <pod-name>                         # Display logs for a specific pod
kubectl logs -f <pod-name>                      # Stream pod logs in real time
kubectl exec -it <pod-name> -- /bin/bash        # Open an interactive shell inside the pod
kubectl delete pod <pod-name>                   # Delete a specific pod
kubectl get pod <pod-name> -o yaml              # Get full pod configuration in YAML format
kubectl get pod -o wide                         # Show pods with extra node and IP details
kubectl port-forward <pod-name> 8080:80         # Forward local port 8080 to port 80 on the pod
kubectl exec -it <pod-name> -- /bin/bash        # Enter container bash shell (bash can also be changed to sh)
kubectl config current-context                  # Show the current context
kubectl config set -context --current --namespace=mealie # Change the current namespace to a different one
kubectl delete pods <pod-name>                  # Delete pod by name
kubectl get deployments.apps                    # Show all deployments
kubectl delete deployments.apps <deployment-name> # Delete a deployment by name
kubectl config get-contexts                     # List all available contexts
kubectl explain pod.spec                        # Get detailed information about pod specifications
kubectl rollout restart deployment ghost-on-kubernetes -n ghost # Restart a deployment to apply changes
kubectl get secrets -A                          # List all secrets on all namespaces
```

## 📦 Deployment Management
```bash
kubectl create deployment hello-world --image=nginx --dry-run=client -o yaml > hello-world-deployment.yaml # Create a deployment YAML file for nginx
kubectl apply -f hello-world-deployment.yaml   # Apply the deployment configuration from a YAML file
kubectl create service nodeport hello-world --tcp=8080:8080 --dry-run=client -o yaml > hello-world-service.yaml
kubectl apply -f hello-world-service.yaml    # Create and apply a NodePort service for the deployment
```

## 👤 User Management
```bash
kubectl config get-users                           # List all users
openssl genrsa -out user.key 2048                  # Generate a private key for user 'user' 
openssl req -new -key user.key -out user.csr -subj "/CN=user/O=readers"
kubectl auth can-i list pods --as johndoe          # Check if user 'johndoe' can list pods
kubectl auth can-i get pods --as johndoe           # Check if user 'johndoe' can get pods
kubectl auth can-i watch pods --as johndoe         # Check if user 'johndoe' can watch pods
kubectl auth can-i delete pods --as johndoe        # Check if user 'johndoe' can delete pods
kubectl config use-context <user@k8s>              # Switch to the context for user 'user'
```

A quick guide on how to create a user [Example Guide](docs/create_user.md)


## 💾 Storage   

```bash
kubectl get pvc                                 # List all Persistent Volume Claims
kubectl get pv                                  # List all Persistent Volumes   
kubectl -n longhorn-system get nodes.longhorn.io # List Longhorn nodes
kubectl -n longhorn-system get volumes.longhorn.io # List Longhorn volumes
kubectl describe pv pvc-3f3b454b-d8ec-4f2b-a3e2-68c059f7a8e5 -o yaml # Describe a specific Persistent Volume in detail
kubectl get storageclasses.storage.k8s.io
```


## 🌐 networking

```bash
kubectl port-forward pods/mealie-7b67c49d9-szrxl 9000 # Temporarily expose an app in a particular port
```


## 🛠️ Troubleshooting

```bash
kubectl edit pod <pod-name>                  # Edit the configuration of a specific pod
watch -n 1 "kubectl get pods" # Run the command continually in the desired interval. Helpful for monitoring pods
kubectl -n kube-system get configmap kubeadm-config -o yaml # View kubeadm configuration
| grep -A 10   # Search and list 10 lines after the match
kubectl run -n <namespace> tmp-shell --rm -it --image=busybox -- /bin/sh # Create a temporary pod for testing network connectivity
kubectl describe node server1 | grep -A5 DiskPressure
kubectl describe node | grep -A5 Conditions
kubectl logs -l name=<label name>
kbuectl drain <node-name> --ignore-daemonsets --delete-local-data # Safely drain a node for maintenance
```

### 🗄️ Backup & Restore etcd Management

```bash
sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd.bakkup  # Backup etcd data


## 👩🏽‍💻 Vim tips
```bash
:%s/test/frontend/g  # Replace all occurrences of 'test' with 'frontend' in the current file
vim -O loadbalancer.yaml ../mealie/service.yaml  # Open multiple files side by side in vim
```

## 📚 Additional Resources

#### 📦 Helm
```bash
helm repo add stable https://charts.helm.sh/stable  # Add the stable Helm chart repository
helm repo update                                 # Update Helm chart repositories
helm install my-release stable/nginx-ingress        # Install the NGINX Ingress Controller using Helm
helm list                                          # List all Helm releases
helm search repo -l cloudflare                     # Search for Cloudflare charts in Helm repositories
helm uninstall my-release                           # Uninstall a Helm release
helm search hub ghost                             # Search for the 'ghost' chart in Helm Hub
helm show values stable/ghost > ghost-values.yaml  # Get default values for the ghost chart and save to a file
helm install my-ghost stable/ghost -f ghost-values.yaml # Install the ghost chart with custom values
helm upgrade prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --values values.yaml # Upgrade an existing Helm release with new values
```

## Helpful linux commands

```bash
find . -name "*:Zone.Identifier" -type f -delete  # Remove Zone.Identifier files created by Windows
du	                                              # Check the size of a folder and its content
rsync	                                            # Sync items between two systems over a network
nc -zv <your-ip-address> <your-port-number> # Check if a specific port is open on a remote server
```

# bashrc shortcuts

```bash
alias dps='docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"'
alias cls='clear'
alias k='kubectl'
source /etc/bash_completion
source <(kubectl completion bash)
complete -F __start_kubectl k
alias kgp='kubectl get pods'
```
