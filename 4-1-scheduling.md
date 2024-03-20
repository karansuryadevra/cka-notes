Without a scheduler:
- use the nodeName field in the spec of the pod (can only be done at creation)
- use `k replace --force -f nginx.yaml` to change a field like nodeName after creation
- or use a `Binding` k8s object

Selectors:
- use `kubectl get pods --selector "app=new,tier=frontend"` 

- use labels for selectors and annotations for integrations

Taints & Tolerations:

- taint is on a node
- toleration is on a pod

Taint: 
- `kubectl taint nodes node-name key=value:taint-effect(NoSchedule,PreferNoSchedule, NoExecute)`
    Eg: 
        `kubectl taint nodes node1 app=blue:NoSchedule`
        To remove the above taint you can just add a "-" to the end like this:
        `kubectl taint nodes node1 app=blue:NoSchedule-`

Toleration: (note the double quotes ("") and how the spec is crafted for the eg above)
    `- spec:
        tolerations:
        - key: "app"
          operator: "Equal"
          value: "blue"
          effect: "NoSchedule"`

Note that the master nodes have a `NoSchedule` taint on them on purpose

Node Selectors:
- uses labels to ensure pods are scheduled on a certain node using `spec.nodeSelector` in the pod definition
- cannot use advanced expressions like OR, NOT

Node Affinity:
    - use operators like "Exists, In" to extend the usage of Node Selectors
        - `Exists` means the label should just exist, value doesnt matter
        - `In` means that the label should be part of a list that the user provides in the config
        - Eg:
           spec:
             affinity:
               nodeAffinity:
                 requiredDuringSchedulingIgnoredDuringExecution:
                   nodeSelectorTerms:
                   - matchExpressions:
                     - key: node-role.kubernetes.io/control-plane
                       operator: Exists
             containers:
             - image: nginx
               name: nginx
               resources: {}
    
               OR
    
            affinity:
              nodeAffinity:
                requiredDuringSchedulingIgnoredDuringExecution:
                  nodeSelectorTerms:
                  - matchExpressions:
                    - key: color
                      operator: In
                      values:
                      - blue    
    
    - types: 
        - requiredDuringSchedulingIgnoredDuringExecution
        - preferredDuringSchedulingIgnoredDuringExecution
    - Change in node Affinity does not make a difference to already scheduled pods due to the "IgnoredDuringExecution" property of the node Affinity

DameonSet:
- Have the same spec as a replicaset except for the kind of the resource (obviously)
- Fastest way to create one is to dry-run a creation of a deployment and then edit the spec to have the kind as daemonset and remove the `spec.replicas` and `spec.strategy fields`

Run `kubectl get events -o wide` to get kubernetes wide events

To make sure your pod is scheduled by a custom scheduler, use the `spec.SchedulerName` field to name your custom scheduler