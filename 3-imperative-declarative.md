Imperative: 
- Tell exactly how to do something
- Egs: 
  ```
    - k run --image=nginx nginx
    - k create deployment -image=nginx nginx
    - k expose deployment nginx --name=nginx-service --port 80    # Create service
    - k edit deployment nginx
    - k scale deployment nginx --replicas=5
    - k set image deployment nginx <container_name>=nginx:1.18
    - k create/replace/delete -f nginx.yaml```
- Better during exam; low extensibility; cant work properly with complex configs


Declarative: Tell what needs to be done, let the software figure out how
- Egs: 
    - k apply -f nginx.yaml   # kubectl understands whether to create/replace/edit on the backend

    the `apply` command adds the "last-applied-configuration" to the annotations of the created resource

Exam tips:

- use `--dry-run=client` to test whether something can be created
- use `-o yaml` to get the resource definition in yaml
- use `kubectl run nginx --image=nginx --dry-run=client -o yaml` to get a pod creation template; OR
- use `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml` to output to a file
- use `kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml` to create a clusterIP service and use pod labels as selectors
- use `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml` to create a nodePort and use pod labels as selectors. However, you cannot specify a nodePort, it will default to a number between 30000 and 32767