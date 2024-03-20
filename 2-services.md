Services allow loose coupling between microservices

Automatic loadbalancing using a random algorithm

apiVersion: v1


NodePort:
- Maps port on node to port on pod
- ports in the spec:
    - targetPort: port on the pod
    - port      : port on the service object itself # Defaults to the same value of targetPort
    - nodePort  : port on the node                  # Defaults to a value between 30000 and 32767


ClusterIP: 
- Creates a virtual IP to allow communication within the cluster
- Is the default type of Service
- Does not need the nodePort port in the `ports` section of the service definition

LoadBalancer:
- For external facing applications

FQDN for services outside your namespace:
   - {service_name.namepsace.svc.cluster.local} 
       - eg: db-service.dev.svc.cluster.local

`namespace` comes under the `.metadata` section in the declaration of the k8s object

Set namespace as default:
    - k config set-context $(kubectl config current-context) --namepace=dev