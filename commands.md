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
kubectl config set -context --current --namespace=mealie # Change the current namespace to a different one
kubectl delete pods <pod-name>                  # Delete pod by name
kubectl get deployments.apps                    # Show all deployments
kubectl delete deployments.apps <deployment-name> # Delete a deployment by name
kubectl config get-contexts                     # List all available contexts
kubectl explain pod.spec                        # Get detailed information about pod specifications
```

## 📦 Deployment Management
```bash
kubectl create deployment hello-world --image=nginx --dry-run=client -o yaml > hello-world-deployment.yaml # Create a deployment YAML file for nginx
kubectl apply -f hello-world-deployment.yaml   # Apply the deployment configuration from a YAML file
kubectl create service nodeport hello-world --tcp=8080:8080 --dry-run=client -o yaml > hello-world-service.yaml
kubectl apply -f hello-world-service.yaml    # Create and apply a NodePort service for the deployment
```

## 💾 Storage

```bash
kubectl get pvc                                 # List all Persistent Volume Claims
kubectl get pv                                  # List all Persistent Volumes   
kubectl -n longhorn-system get nodes.longhorn.io # List Longhorn nodes
kubectl -n longhorn-system get volumes.longhorn.io # List Longhorn volumes
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
```

## 👩🏽‍💻 Vim tips
```bash
:%s/test/frontend/g  # Replace all occurrences of 'test' with 'frontend' in the current file
vim -O loadbalancer.yaml ../mealie/service.yaml  # Open multiple files side by side in vim
```

## 📚 Additional Resources

#### 📦 Helm
```bash
helm repo add stable https://charts.helm.sh/stable  # Add the stable Helm chart repository
helm install my-release stable/nginx-ingress        # Install the NGINX Ingress Controller using Helm
helm list                                          # List all Helm releases
helm uninstall my-release                           # Uninstall a Helm release
helm search hub ghost                             # Search for the 'ghost' chart in Helm Hub
helm show values stable/ghost > ghost-values.yaml  # Get default values for the ghost chart and save to a file
helm install my-ghost stable/ghost -f ghost-values.yaml # Install the ghost chart with custom values
helm upgrade prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --values values.yaml # Upgrade an existing Helm release with new values
```

## Helpful linux commands

```bash
find . -name "*:Zone.Identifier" -type f -delete  # Remove Zone.Identifier files created by Windows
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