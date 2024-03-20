- While testing the Network Namespaces, if you come across issues where you can't ping one namespace from the other, make sure you set the NETMASK while setting IP Address. ie: 192.168.1.10/24


`ip -n red addr add 192.168.1.10/24 dev veth-red`

Another thing to check is FirewallD/IP Table rules. Either add rules to IP Tables to allow traffic from one namespace to another. Or disable IP Tables all together (Only in a learning environment).

- In docker, you can create a bridge network that is created by default. Its address is 172.17.0.0 called docker0 if you run the `ip link`

- Helpful commands to keep handy:
  - `ip a | grep -B2 192.5.91.6`
  - `ip link` --> gives you info about the bridges and ports
  - `ip addr`
  - `ip addr add 192.168.1.10/24 dev eth0`
  - `ip route`
  - `ip route show default gateway`
  - `ip route add 192.168.10/24 via 192.168.2.1`
  - `cat /proc/sys/net/ipv4/ip_forward`
  - `arp`
  - `netstat -plnt`
  - `netstat -anp | grep etcd` --> Shows all the connections to a service
  - `route`

- To setup pod networking we could run a script on each node like: 
  - ### Create veth pair
  - `ip link add ...`
  
  - ### Attach veth pair
  - `ip link set ...`
  - `ip link set ...`
  
  - ### Assign IP Address
  - `ip -n <namespace> addr add ...`
  - `ip -n <namespace> route add ...` 

  - ### Bring up interface
  - `ip -n <namespace> link set ...`
- ^ These steps would be part of the `ADD` section of a config script for a CNI

- Run `ps -aux | grep kubelet` to get the options used to run kubelet
- Look for `--cni-bin-directory` and `--network plugin`
- If you `ls` the `cni-conf-dir` and check out the file, it will point you to the actual config file
- The CNI binaries are located under `/opt/cni/bin` by default.
- `ls /etc/cni/net.d/` and identify the name of the CNI plugin.

- kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
- Use the `ipam` section of the cni plugin config at `etc/cni/net.d/net-script.conf` to setup your IP addresses



# Service Networking

- When a srevice is created, it is assigned an IP from a pre-existing list of available IPs
- The magic is that each kube-proxy on each node adds forwarding rules that state requests coming to the IP of the service need to be routed to the IP of the respective pods (remember kube-proxy and kubelet run on all nodes)
- This is done via `iptables` by default by the kube-proxy
- The CIDR range for pods and services should not overlap 
- You can check out the range for services by running `ps aux | grep kube-api-server` and then look for the `--service-cluster-ip-range` to get an idea
- Run `ip addr show <sni_name>` and look at the range there to get the range of the pods


# DNS
- kube-dns has a table that maps service names to their IPs and also takes into account the namespace
- corends creates a service whose IP is the nameserver in every pods `/etc/resolve.conf`

Egs:
    - Service FQDN: `servicename.namespace.svc.cluster.local`
    - Pod FQDN: `podip_replace_dots_with_dashes.namespace.pod.cluster.local`
  - The root domain is `cluster.local` for short you can't use just `cluster` it has to be `cluster.local`

# Ingress:
- L7 load balancer. Endpoint routing and SSL enforcement
- Deploy using a Ingress Controller (nginx, haproxy, traefik, istio) and Ingress Resources
- We can create an Ingress resource from the imperative way like this:-
    - Format - `kubectl create ingress <ingress-name> --rule="host/path=service:port"`
    - Example - `kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"`
- `k get ingress --all-namespaces` to get the Ingress Resources
- Ingress resources are namespaced
- Eg:
    - ```
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: test-ingress
          namespace: critical-space
          annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
            nginx.ingress.kubernetes.io/ssl-redirect: "false"
        spec:
          rules:
          - http:
              paths:
              - path: /pay
                pathType: Prefix
                backend:
                  service:
                   name: pay-service
                   port:
                    number: 8282
    ```
- Use the `k create ingress --help` command to give you quick examples of imperative commands to create an ingress resource

# Creating an Ingress Controller
- Create a namespace for it
- Create an empty configmap
- Create 2 service accounts: ingress-nginx, ingress-nginx-admission
- Create roles, cluterroles, rolebindings, clusterrolebindings for each service account