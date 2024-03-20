If a worker node doesnt have an associated master, you can still create pods on it by using the kubelet

Place the pod definition spec file in the `/etc/kubernetes/manifests` directory on the worker node.

Notes: 
1) You can only create pods using this method
2) Modifying/deleting the pod config file has the same effect on the pod that exists on the node

These are called Static Pods
The kubelet only understands Pods, hence you can only create pods

You can use docker commands to see the containers (there's no kube-apiserver to take your kubectl requests)

KubeADM sets up control-plane components as static pods so that the pods cannot be modified externally. Only by modifying the files can a change be made

How to tell if a pod is a static pod:
1) the pod name will be suffixed with the node name
2) If you look at the YAML of the pod, it will have a `ownerReferences` block which will have the `kind: Node` which means that the controller of the pod is the node itself

To find where the static pod configs are stored you can:
1) Check the kubelet config in /var/lib/kubelet/config.yaml and see the `staticPodPath`
2) Or run `ps -aux | grep /usr/bin/kubelet` and then look at the `--config` flag in the commands and get the kubelet path. From there do the same as  1

