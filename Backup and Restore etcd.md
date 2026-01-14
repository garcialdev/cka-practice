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