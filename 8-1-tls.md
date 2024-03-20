.key = private key
.crt = certificate


Servers: 
- kube-api server
- etcd server
- kubelet server

Clients:
- Admins
- kube-scheduler
- kube-controller-manager
- kube-proxy
- kube api-server is a client for the etcd server and the kubelet server 