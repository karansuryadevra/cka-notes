The time it takes for the kube controller manager to reassign a pod from a dead node to another alive node is called the PodEvictionTimout # default is 5 mins

`kubectl drain node-1` --> recreates pods from node-1 to other nodes and marks the node Unschedulable
`kubectl uncordon node-1` --> makes the node Schedulable
`kubectl cordon node-1` --> makes the node Unschedulable

Upgrade steps:
1) upgrade master --> Nothing new can be done, everything existing on worker nodes will work as usual
2) upgrade workers --> Can upgrade all at a time or one at a time, by shifting loads from the nodes that are down to other nodes or you can add a new node with the new version of k8s and remove the old nodes one by one


`kubeadm upgrade plan`  # kubeadm doesnt upgrade kubelet
- Upgrade master node:
 ```
root@controlplane:~# kubectl drain controlplane --ignore-daemonsets
root@controlplane:~# apt update
root@controlplane:~# apt-get install kubeadm=1.27.0-00
root@controlplane:~# kubeadm upgrade plan v1.27.0
root@controlplane:~# kubeadm upgrade apply v1.27.0
root@controlplane:~# apt-get install kubelet=1.27.0-00
root@controlplane:~# systemctl daemon-reload
root@controlplane:~# systemctl restart kubelet
root@controlplane:~# kubectl uncordon controlplane
```





Details:
1) upgrade kubeadm first: `apt-get upgrade -y kubeadm=<version>`
2) `kubeadm upgrade apply v<version>`
3) `k get nodes`
4) `apt-get upgrade -y kubelet=<version>`
5) systemsctl restart kubelet
6) Master node done!
7) Drain a worker node
8) upgrade kubeadm and kubelet
9) `kubeadm upgrade node config --kubelet-version v<version>`
10) `sudo systemctl daemon-reload && sudo systemctl restart kubelet`
11) uncordon node
12) Repeat 7-12 for all worker nodes 1 by 1

All in all, its probably best to follow the steps in the k8s docs. They're straight forward for upgrading both the master and the worker nodes

# ETCD

Backup ETCD:

- `ETCDCTL_API=3 etcdtl snapshot save snapshot.db`
- `ETCDCTL_API=3 etcdtl snapshot status snapshot.db`
- `service kube-apiserver stop`
- `ETCDCTL_API=3 etcdtl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup`
- Since the etcd may be installed as a static pod, you will have to edit `/etc/kubernetes/manifests/etcd.yaml` to take on your new `--data-dir`
- You need to specify cert args to these etcdctl commands, might as well look it up on the k8s website
- We have now restored the etcd snapshot to a new path on the controlplane - /var/lib/etcd-from-backup, so, the only change to be made in the YAML file, is to change the hostPath for the volume called etcd-data from old directory (/var/lib/etcd) to the new directory (/var/lib/etcd-from-backup).
```
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```

With this change, /var/lib/etcd on the container points to /var/lib/etcd-from-backup on the controlplane (which is what we want).
- 

    Note 1: As the ETCD pod has changed it will automatically restart, and also kube-controller-manager and kube-scheduler. Wait 1-2 to mins for this pods to restart. You can run the command: watch "crictl ps | grep etcd" to see when the ETCD pod is restarted.

    Note 2: If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

    Note 3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.

- compulsory flags for the etcdctl commands if you have TLS enabled (you can look for the etcd process using ps aux to get the values for the flags):
```
--cacert  [--trusted-ca-file]                                              verify certificates of TLS-enabled secure servers using this CA bundle

--cert    [--cert-file]                                                identify secure client using this TLS certificate file

--endpoints=[127.0.0.1]:2379 or "https://127.0.0.1:2379"         This is the default as ETCD is running on master node and exposed on localhost 2379.

--key     [--key-file]                                                 identify secure client using this TLS key file
```

- ETCD can be installed in 2 ways: 
- 1) Stacked ETCD: It is installed on the controlplane of the cluster
- 2) External ETCD: It is running as a separate entity on the node # You wont see the etcd pod in the kube-system namespace