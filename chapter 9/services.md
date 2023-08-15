# Services

Service provide an abstract way to expose an application running as a set of Pods, for clients to access them.

Services route traffic to Pods in a load-balanced fashion.

Endpoints = backend entities services route traffic to. Each Pod will have an endpoint associated with the Service.


## Configuration

 - type
 - selector
 - ports

### Managing

Display the IPs/Ports for service:
`kubectl get endpoints <svcname>`



## Service Types

Service types:
 - ClusterIP
 - NodePort
 - LoadBalancer
 - ExternalName (not in CKA)

### ClusterIP

Purpose: Expose applications **inside** the cluster network.

The default, if `type` is not specified.

### NodePort

Purpose: Expose applications **outside** the cluster.

Configs:

- spec.ports.nodePort: will expose this port on EVERY node of the cluster. (optional - will assign random port if not specified)


### LoadBalancer

Purpose: Expose apps **outside** the cluster. Uses cloud native load balancers.

## Lab

They have a backend database of user information and a web frontend, both of which are managed using deployments. Both of these applications are also still being developed and are currently just using simple Nginx containers for testing. You have been asked to set up appropriate Kubernetes Services for these application components.

The user database is a backend service that should only be accessible by other components within the cluster. The web frontend needs to be accessible by users outside the cluster. Locate the existing deployments and create the necessary Services to expose them. There is an existing Pod called `busybox` which you can use to test Services.

## Service DNS

DNS assigns hostnames to Services:

FQDN: `service-name.namespace-name.svc.cluster.local`

DN Within Namespace: `service-name`

### Testing

Get FQDN:

`kubectl exec pod-svc-test -- nslookup <clusterip>`

### Lab

Your developers aren't quite sure how Kubernetes DNS works, and they are having trouble reaching some of their Services as a result. They have asked you to perform some tests that will help confirm and clarify for them how Kubernetes DNS behaves.

A Pod called `busybox` already exists in the `web` namespace. You can use this Pod to perform your tests. Save the output of your tests to some files, so that the developers can see the results.

```
kubectl exec busybox -n web -- nslookup web-frontend
kubectl exec busybox -n web -- nslookup web-frontend.web.svc.cluster.local
```

#### Perform an Nslookup for a Service in the Same Namespace

Use the `busybox` Pod in the `web` namespace to perform an nslookup on the `web-frontend` Service. Use both the short Service name and its fully qualified domain name, and save all of the results to `~/dns_same_namespace_results.txt`.

```
kubectl exec busybox -n web -- nslookup user-db
kubectl exec busybox -n web -- nslookup user-db.data.svc.cluster.local
```

## Ingress

Ingress manages external access to Services.

More functionality than NodePort Services:
 - SSL termination
 - Advanced load balancing
 - name-based virtual hosting

Must install Ingress controllers before using.

Configs:
 - Routing Rules - which incoming requests apply
 - Paths - Requests matching a path will be routed to associated backend
 - Backends - Service name and port

> If a Service uses a Named port, can use this instead of port numbers.

### Lab

One of these applications, a web authentication service, will need to be accessible by external clients using the BeeBox mobile app. A Deployment has been created to represent this Service (the app is still in early stages, so it is just running Nginx containers for now).

Create a ClusterIP service that will expose the web-auth Deployment. Then, create an Ingress which maps requests with the path /auth to the Service.

For now, you do not need to worry about installing any Ingress controllers.

#### Create a Service to Expose the web-auth Deployment

Create a service that will expose the Pods from the `web-auth` Deployment, located in the `default` namespace. The Deployment's Pods publish port 80. Expose port 80 on the Service itself as well.

```
$ kubectl get deployments -o wide
NAME       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
web-auth   2/2     2            2           93m   nginx        nginx:1.19.1   app=web-auth


apiVersion: v1
kind: Service
metadata:
  name: svc-web-auth
spec:
  selector:
    app: web-auth
  ports:
  - name: http
    protocol: TCP 
    port: 80
    targetPort: 80

$ kubectl get endpoints svc-web-auth
NAME           ENDPOINTS                             AGE
svc-web-auth   192.168.194.65:80,192.168.194.66:80   5s
```

#### Create an Ingress That Maps to the New Service

Create an Ingress that maps to the new Service. This Ingress should route requests with the path `/auth` to port 80 on the Service.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web-auth
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: svc-web-auth
            port:
              name: http
```