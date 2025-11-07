# 📘 Kubernetes Command Cheat Sheet

A personal collection of useful `kubectl` and Kubernetes-related commands for quick reference and study.
Feel free to fork or clone this file to your GitHub repo!

---

## 🚀 Pod Management

```bash
kubectl get pods                          # List all pods in the current namespace
kubectl get pods -A                       # List all pods across all namespaces
kubectl describe pod <pod-name>           # Show detailed information about a specific pod
kubectl logs <pod-name>                   # Display logs for a specific pod
kubectl logs -f <pod-name>                # Stream pod logs in real time
kubectl exec -it <pod-name> -- /bin/bash  # Open an interactive shell inside the pod
kubectl delete pod <pod-name>             # Delete a specific pod
kubectl get pod <pod-name> -o yaml        # Get full pod configuration in YAML format
kubectl get pod -o wide                   # Show pods with extra node and IP details
kubectl port-forward <pod-name> 8080:80   # Forward local port 8080 to port 80 on the pod
kubectl exec -it <pod-name> -- /bin/bash  # Enter container bash shell (bash can also be changed to sh)
```

## 🛠️ Troubleshooting

```bash
watch -n 1 "kubectl get pods" # Run the command continually in the desired interval. Helpful for monitoring pods
```
