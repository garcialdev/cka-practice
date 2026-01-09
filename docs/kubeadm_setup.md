# Kubernetes Installation Guide for Ubuntu 24.04


I've had difficulty setting up my new vm with all the neccessary things required to run kubernetes. 
This guide covers installing Kubernetes (kubeadm, kubelet, kubectl) and containerd on a fresh Ubuntu server.

## Prerequisites

- Ubuntu 24.04 (Noble) server
- Sudo/root access
- At least 2GB RAM
- 2 CPUs minimum

## Step 1: System Preparation

Login to your server and let's run the following commands.

Update the system and disable swap (required for Kubernetes):

```bash
sudo apt-get update
sudo apt-get upgrade -y

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Step 2: Install Container Runtime (containerd)

### Add Docker's Repository

```bash
# Install prerequisites
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

### Install containerd

Check available versions first to avoid 404 errors:

```bash
apt-cache madison containerd.io
```
I had issues with the latest version so if there is a problem try a previous version. I installed 1.7.29 since 2.2.1 was giving me problems. 

```bash
# Option 1: Install latest stable 1.7.x version
sudo apt-get install -y containerd.io=1.7.29-1~ubuntu.24.04~noble

# Option 2: Install 2.2.0 if preferred
sudo apt-get install -y containerd.io=2.2.0-2~ubuntu.24.04~noble
```

### Configure containerd for Kubernetes

```bash
# Generate default config
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# Enable SystemdCgroup (required for Kubernetes)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```

Verify CRI is working:

```bash
sudo crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock version
```

## Step 3: Install Kubernetes Components

### Enable required kernel modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### Configure sysctl parameters

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### Add Kubernetes Repository

```bash
# Add Kubernetes GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
```

### Install kubeadm, kubelet, and kubectl

```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Step 4: Initialize Kubernetes Cluster (Control Plane Only)

On the control plane node:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

After initialization completes, set up kubectl access:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 5: Install Pod Network Add-on

Install Flannel (or another CNI):

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Step 6: Verify Installation

```bash
kubectl get nodes
kubectl get pods -A
```

Your node should show as "Ready" after a minute or two.

## Troubleshooting

### containerd 404 Error
If you get a 404 error during installation, check available versions:
```bash
apt-cache madison containerd.io
```
Then install a specific version that exists on the download server.

### CRI Runtime Error
If kubeadm complains about CRI runtime, ensure containerd config has `SystemdCgroup = true`:
```bash
grep SystemdCgroup /etc/containerd/config.toml
sudo systemctl restart containerd
```

### Node Not Ready
Check pod network is running:
```bash
kubectl get pods -n kube-flannel
kubectl get pods -n kube-system
```

## Joining Worker Nodes

On worker nodes, complete Steps 1-3, then use the join command from the control plane's init output:

```bash
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

If you lost the join command:

```bash
kubeadm token create --print-join-command
```

## Useful Commands

```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes
kubectl get pods -A

# Reset cluster (if needed)
sudo kubeadm reset
sudo rm -rf /etc/cni/net.d
sudo rm -rf $HOME/.kube/config
```