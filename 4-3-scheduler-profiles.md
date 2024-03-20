Scheduler config:
```
apiVersion: kubescheduler.config.k8s.io/v1
 kind: KubeSchedulerConfiguration
 profiles:
 - schedulerName: my-scheduler
 ```

Scheduling Steps:
1) Pods are put into a `Scheduling Queue` based on their `spec.priorityClassName`, descending order of priority  # PrioritySort plugin
2) Nodes are filtered out based on whether they have enough resources to host the pod                           # NodeResourcesFit, NodeName,   NodeUnschedulable plugin
3) The nodes are then scored based on the resources the node will have after scheduling the pod on it           # NodeResourcesFit, ImageLocality plugin
4) The pod is then bound to the node                                                                            # DefualtBinder plugin

To prevent race conditions for schedulers scheduling pods on the same nodes, you can use schedulerProfiles and customize each scheduler's plugins using one config file
