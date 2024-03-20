- Create a network policy to dictate what kind of traffic can flow to and from a pod.
- By default all pods can talk to and listen to all other pods

Eg: Policy that allows my db pod to accept traffic from my api pod on port 3306
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    ingress:
    - from:
      - podSelector:
          matchLabels:
            name: api-pod
      ports:
      - protocol: TCP
        port: 3306

```