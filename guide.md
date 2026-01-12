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
The output indicates that the etcd snapshot has been saved to the file /opt/etcd.bak.
```