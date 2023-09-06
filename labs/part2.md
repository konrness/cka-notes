# Lab - Part 2

This question uses the acgk8s cluster. After logging in to the exam server, switch to the correct context with the command kubectl config use-context acgk8s.

Each of the objectives represents a task which you will need to complete using the available cluster and server(s). Read each objective carefully and complete the task specified.

For some objectives, you may need to ssh into other nodes or servers from the exam server. You can do so using the hostname/node name (i.e., ssh acgk8s-worker1).

Note: You cannot ssh into another node, or use kubectl to connect to the cluster, from any node other than the root node. Once you have completed the necessary tasks on a server, be sure to exit and return to the root node before proceeding.

If you need to assume root privileges on a server, you can do so with sudo -i.

You can run the verification script located at /home/cloud_user/verify.sh at any time to check your work!

### Edit the Web Frontend Deployment to Expose the HTTP Port

In the web namespace, there is a deployment called web-frontend.

Edit this deployment so that the containers within its Pods expose port 80.

```
$ kubectl edit deployment web-frontend -n web


...
    spec:
      containers:
      - image: nginx:1.14.2
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
...

```

### Create a Service to Expose the Web Frontend Deployment's Pods Externally

Create a service called web-frontend-svc in the web namespace. This service should make the Pods from the web-frontend deployment in the web namespace reachable from outside the cluster.

External entities should be able to reach the service by contacting any node in the cluster on port 30080.

```
apiVersion: v1
kind: Service
metadata:
  name: web-frontend-svc
  namespace: web
spec:
  type: NodePort
  selector:
    app: web-frontend
  ports:
    - port: 80
      nodePort: 30080

$ kubectl get endpoints web-frontend-svc -o wide -n web
NAME               ENDPOINTS                                          AGE
web-frontend-svc   192.168.19.1:80,192.168.19.2:80,192.168.228.7:80   8m36s
```

### Scale Up the Web Frontend Deployment

Scale the web-frontend deployment in the web namespace up to 5 replicas.

```
$ kubectl scale --replicas=5 deployment/web-frontend -n web
deployment.apps/web-frontend scaled
```

### Create an Ingress That Maps to the New Service

Create an Ingress called web-frontend-ingress in the web namespace that maps to the web-frontend-svc service in the web namespace. The Ingress should map all requests on the / path.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-frontend-ingress
  namespace: web
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-frontend-svc
            port:
              number: 80