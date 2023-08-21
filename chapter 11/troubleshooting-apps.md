# Troubleshooting Apps

## Check Pod Status

`kubectl get pods`

`kubectl describe pod [podname]`

## Run Commands in Containers

`kubectl exec [podname] -c [containername] -- [command]`

**Interactive shell:**

`kubectl exec [podname] -c [containername] --stdin --tty -- /bin/sh`

## Check Container Logs

Container's log contains everything written to stdout and stderr.

`kubectl logs [podname] -c [containername]`

## Lab

Your company, BeeBox, is building some applications for Kubernetes. Your developers have recently deployed an application to your cluster, but it is having some issues.

A set of Pods managed by the `web-consumer` deployment regularly make requests to a service that provides authentication data. Your developers are reporting that the containers are not behaving as expected.

Your task is to look at the application in question, determine the problem, and fix it.

### Identify What is Wrong with the Application

Examine the `web-consumer` deployment and its Pods. The deployment resides in the `web` namespace. Determine what may be going wrong.

```
$ kubectl get pods -n web
NAME                            READY   STATUS    RESTARTS   AGE
web-consumer-84fc79d94d-hmqrv   1/1     Running   0          9m35s
web-consumer-84fc79d94d-mzmjr   1/1     Running   0          9m35s

$ kubectl logs web-consumer-84fc79d94d-hmqrv -n web
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'

$ kubectl describe pod  web-consumer-84fc79d94d-hmqrv -n web
...
    Command:
      sh
      -c
      while true; do curl auth-db; sleep 5; done
...
```

```
$ kubectl get services --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
data          auth-db      ClusterIP   10.101.120.136   <none>        80/TCP                   12m
default       kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP                  13m
kube-system   kube-dns     ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   13m
```

There is no `auth-db` service in the `web` namespace. It is in the `data` namespace.


### Fix the Problem

Make changes to objects in the cluster to fix the problem and allow the `web-consumer` deployment's Pods to communicate with the service successfully.

The correct hostname for the `auth-db` service is `auth-db.data.svc.cluster.local`

```
$ kubectl edit deployment web-consumer -n web
```

```
$ kubectl get pods -n web
NAME                            READY   STATUS        RESTARTS   AGE
web-consumer-746567899c-c5bgm   1/1     Running       0          23s
web-consumer-746567899c-fpbg8   1/1     Running       0          25s
web-consumer-84fc79d94d-hmqrv   1/1     Terminating   0          15m
web-consumer-84fc79d94d-mzmjr   1/1     Terminating   0          15m
cloud_user@k8s-control:~$ kubectl logs web-consumer-746567899c-c5bgm -n web
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   102k      0 --:--:-- --:--:-- --:--:--  149k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
```