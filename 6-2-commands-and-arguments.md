The difference between `CMD` and `ENTRYPOINT` in the container creation in docker is: 

- CMD["sleep", "5"]    --> If we want to override the arg 5 and make it 10, it is a pain
- ENTRYPOINT["sleep"]  --> Simply pass the arg in the docker run command to append to the entry point like: `docker run test 10`

Use both together to have a default: 
ENTRYPOINT["sleep"]
CMD["5"]  # Executes if no arg is passed to docker run

In a pod definiton spec: 
Use `spec.containers.args` to override the CMD value of the image
Use `spec.containers.command` to override the ENTRYPOINT value of the image

OR you can do:
```
spec:
  containers:
    command:
    - "sleep"
    - "5000"
```    

Use `spec.containers[].env[]` to setup `name: "" value: ""` pairs of env variables