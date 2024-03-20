1 CPU = 1 unit that is offered by the cloud provider (1vCPU in GCP etc.)
You cannot go lower than 0.1 CPU which is equivalent to 1m

The request is what the scheduler looks at when it is scheduling the pod

Eg:
    spec:
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "10Gi"
          cpu: 5

Pod can go over its CPU limit and it gets throttled. But if it keeps going over its memory limit it will be OOMKilled

By default, requests=limits

Use a LimitRange object to set default limits:
- Eg for CPU:
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-resource-constraint
    spec:
      limits:
      - default:
          cpu: 500m
        defaultRequest:
          cpu: 500m
        max:
          cpu: "1"
        min:
          cpu: "1"
        type: Container

Use a ResourceQuota to set resource constraints across a namespace