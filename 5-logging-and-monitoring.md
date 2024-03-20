# Monitoring

The metrics server is used to store the metrics in-memory, if you want disk storage you can use prometheus etc.

The `cAdvisor` component of the kubelet is responsible for exposing the pod-level metrics to the metrics server

- `kubectl top node`
- `kubectl top pod`


# Logging
- `kubectl logs -f <pod_name>`