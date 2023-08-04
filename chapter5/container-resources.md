# Container Resources

## Resource Requests

Only used for scheduling, containers are allowed to use more than the request

Memory measured in bytes

CPU measured in milli-CPUs (1/1000th CPU)

## Resource Limits

Stops containers from using more resources than they should. The container runtime enforces the limits. Some may enforce by terminating the container.

e.g. Docker: will throttle based on CPU limits, and will kill for memory limits.



### Example

```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

