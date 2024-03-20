# ConfigMaps
## Imperative:
```
kubectl create configmap --from-literal=app=blue \
    --from-literal=version=v2
```

OR

`kubectl create configmap --from-file=<path_to_file>`

## Declarative

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP: blue
    VERSION: v2
```

To inject CM into pod use: `spec.containers[].envFrom.configMapRef` which has a `name` field # This will ensure all the variables are injected
Or you can also use `spec.containers[].env.valueFrom.configMapRef` like this to inject only select values: 
```
spec:
  containers:
  - name: <name> 
    env:
      # Define the environment variable
      - name: SPECIAL_LEVEL_KEY
        valueFrom:
          configMapKeyRef:
            # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
            name: special-config
            # Specify the key associated with the value
            key: color
```
Use volumes to inject multiple values 

# Secrets
## Imperative:
```
kubectl create secret generic  <secret-name> --from-literal=app=blue \
    --from-literal=version=v2
```

OR

`kubectl create secret generic <secret-name> --from-file=<path_to_file>`


## Declarative

- Get base64 encoded values of your secrets by running:
  - `echo -n "test" | base64`

```
apiVersion: v1
kind: Secret
metadata:
    name: app-secret
data:
    APP: <base64_encoded_secret>
    VERSION: <base64_encoded_secret>
```

Run `kubectl get secret <secret_name> -o yaml` to get the base64 encoded values of the secrets

- To inject all the values into a pod: 

```
spec:
  containers:
    - name: test
      envFrom:
        - secretRef:
            name: app-secret
```

- To inject 1 value into a pod: 
```
spec:
  containers:
    - name: test
      env:
        - valueFrom:
            secretKeyRef:
              name: app-secret
              key: test_value
```

If you mount a secret as a volume to a pod, the pod is created with the same number of secret files as the number of secrets that are stored in k8s object

## Secrets are not encrypted, they are just encoded!!