# 🗄️ How to backup and restore etcd data


Etcd is deployed as a Pod in the kube-system namespace. The name of the Pod is etcd-controlplane:

```bash
$ kubectl get pods  -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
etcd-controlplane                      1/1     Running   0          3m56s
Inspect the version by describing the Pod. In the output below, you will find that the version is 3.5.1-0:
```

```bash
$ kubectl describe pod etcd-controlplane -n kube-system
Containers:
  etcd:
    Container ID:  containerd://ffaff8dcbf40eacbcfa07ce335e57b05057ad22bb8efde0b68e679ffc193300a
    Image:         registry.k8s.io/etcd:3.5.15-0
```

The same describe command reveals the configuration of the etcd service. Look for the value of the option --listen-client-urls for the endpoint URL. In the output below, the host is localhost and the port is 2379. The server certificate is located at /etc/kubernetes/pki/etcd/server.crt defined by the option --cert-file. The CA certificate can be found at /etc/kubernetes/pki/etcd/ca.crt specified by the option --trusted-ca-file:

```bash
$ kubectl describe pod etcd-controlplane -n kube-system

Containers:
  etcd:
    ...
    Command:
      etcd
      ...
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

Use the etcdctl command to create the backup with version 3 of the tool. For a good starting point, copy the command from the official Kubernetes documentation. Provide the mandatory command line options --cacert, --cert, and --key. The option --endpoints is not needed, as we are running the command on the same server as etcd. After running the command, the file /opt/etcd.bak has been created:

```bash
$ sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd.bak
Snapshot saved at /opt/etcd.bak
```

The output indicates that the etcd snapshot has been saved to the file /opt/etcd.bak.


# Restoring etcd from Backup

To restore etcd from the backup, use the etcdutl snapshot restore command. At a minimum, provide the --data-dir command line option. Here, we are using the data directory /var/bak. After running the command, you should be able to find the restored backup in the directory /var/bak:

$ sudo ETCDCTL_API=3 etcdutl --data-dir=/var/bak snapshot restore /opt/etcd.bak
Edit the YAML manifest of the etcd Pod, which can be found at /etc/kubernetes/manifests/etcd.yaml. Change the value of the attribute .spec.volumes.hostPath with the name etcd-data from the original value /var/lib/etcd to /var/bak:

```bash
spec:
  volumes:
  ...
  - hostPath:
      path: /var/bak
      type: DirectoryOrCreate
    name: etcd-data
```

Restart the kubelet to ensure that those changes to the etcd database are picked up:

$ sudo systemctl restart kubelet.service
The etcd-controlplane Pod will be recreated and points to the restored backup directory. It might take a couple of seconds before the command returns:

```bash
$ kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
etcd-controlplane                      1/1     Running   0          3m56s
```

In case the Pod doesn't transition into the Running status, try to delete it manually with the command kubectl delete pod etcd-controlplane -n kube-system.

#### Working code for my node

```bash
 kubectl exec -n kube-system etcd-k1 -- sh -c '
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/etcd.bak \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
'
```

✅ I got this successful message after:

```bash
Snapshot saved at /var/lib/etcd/etcd.bak
```

***
❌ Explanation of why the command

```bash
$ sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd.bak
```
❌ did not work


##### 1️⃣ What kubectl exec actually does

When you run:

```bash
kubectl exec POD -- some-command
```

Kubernetes does NOT run a shell by default.
It directly tries to execute exactly what comes after -- as a binary inside the container.

So Kubernetes behaves like:

```bash
execve("some-command", ["some-command", ...])
```

No shell.
No variable expansion.
No pipes.
No VAR=value handling.

##### 2️⃣ Why this did NOT work earlier

You tried something like:
```bash
kubectl exec ... -- ETCDCTL_API=3 etcdctl snapshot save ...
```

Kubernetes interpreted this as:

```bash
Run a binary named: ETCDCTL_API=3
```

Which obviously doesn’t exist.

Hence the error:
```bash
exec: "ETCDCTL_API=3": executable file not found in $PATH
```

💡 Key point:
VAR=value command is shell syntax, not Linux process syntax.

##### 3️⃣ Why your working command does work

Your successful command:

```bash
kubectl exec -n kube-system etcd-k1 -- sh -c '
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/etcd.bak \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
'
```
What actually happens

Kubernetes executes one real binary:

```bash
sh
```

sh receives the -c flag, meaning:

“Run the following string as shell code”

Inside the shell:
ETCDCTL_API=3 is correctly treated as an environment variable

etcdctl is found in $PATH

Arguments are parsed normally

So the shell does the job Kubernetes won’t do on its own.

##### 4️⃣ Why this is especially important for etcd containers

The etcd image (registry.k8s.io/etcd) is:

Minimal

No systemd

No login shell

No interactive environment

Often distroless or near-distroless

That means:

You must be very explicit

You only get what’s already inside the container

Shell parsing must be deliberate

##### 5️⃣ Two valid patterns (remember these)
✅ Pattern A — Use a shell
```bash
kubectl exec POD -- sh -c 'VAR=value command args'
```

✔ Best when you need:

env vars

pipes

redirects

multiple commands

✅ Pattern B — Use env (no shell)
```bash
kubectl exec POD -- env VAR=value command args
```

✔ Best when:

container has no shell

you want maximum reliability

For etcd both work, but env is technically cleaner.

##### 6️⃣ Mental model (this clicks fast)

Think of it like this:

Context	Who parses the command?
SSH	Shell
Local terminal	Shell
kubectl exec	❌ Nobody
kubectl exec -- sh -c	Shell

If no shell runs, shell syntax won’t work.

##### 7️⃣ Why this mattered for your PVC / Calico saga

You’ve been dealing with:

Minimal containers

Broken networking

kubeadm static pods

CSI dependencies

Calico cert issues

Understanding this distinction explains:

Why binaries “don’t exist”

Why env vars fail

Why some containers feel “locked down”

This knowledge is cluster-admin-level knowledge.

✅ Final takeaway (bookmark this)

kubectl exec does not run a shell unless you explicitly start one.

Once that clicks, 80% of exec-related errors disappear.