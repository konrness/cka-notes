# Deployments

Definition of desired state for a `ReplicaSet`.

Config Options:
 - `replicas` - # of pods
 - `selector` - a label selector to identify replica pods managed by deployment
 - `template` - the template for each pod

Use Cases:
 - Easily scale an app up/down in number of replicas
 - Perform rolling updates to deploy a new software version
 - Roll back to a previous software version

## Scaling Deployments

Useful for horizontal scaling, increase/decrease quantity of pod. Defined by `replicas` setting.

Options:

 1. Apply/edit deployment's `spec.replicas` in YAML
 2. `kubectl scale deployment.v1.apps/my-deployment --replicas=3`

## Rolling Updates with Deployments

- Rolling Update - gradually add new pods and delete old pods
- Rollback - return to previous state

How to execute rolling updates:
1. Apply/edit deployment YAML
2. `kubectl set image deployment/my-deployment <containername>=<image:tag> --record`

### Check status of rollout:

`kubectl rollout status deployment.v1.apps/my-deployment`

### Get rollout history

`kubectl rollout history deployment.v1.apps/mydeployment`

Lists the revisions and change-cause's.

### Roll Back

`kubectl rollout undo deployment.v1.apps/my-deployment`

`kubectl rollout undo deployment.v1.apps/my-deployment --to-revision=2`

# Lab

One of these applications is a simple web server. It is being managed in Kubernetes using a deployment called `beebox-web`. Unfortunately, there are some problems with the app, and it is performing poorly under large user load.

Two steps will need to be taken to fix this issue. First, you will need to deploy a newer version of the app (`1.0.2`) that contains some performance improvements from the developers. Second, you will need to scale the app deployment, increasing the number of replicas from `2` to `5`.

## Update the App to a New Version of the Code

Perform a rolling deployment on the `beebox-web` deployment to deploy version `1.0.2`.

`kubectl describe deployment beebox-web`

Container name is `web-server`. Current image is `acgorg/beebox-web:1.0.1`.

`kubectl set image deployment/beebox-web web-server=acgorg/beebox-web:1.0.2 --record`

```
$ kubectl rollout status deployment/beebox-web
deployment "beebox-web" successfully rolled out

$ kubectl rollout history deployment/beebox-web
deployment.apps/beebox-web
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/beebox-web web-server=acgorg/beebox-web:1.0.2 --record=true
```

## Scale the App to a Larger Number of Replicas

Increase the number of replicas in the `beebox-web` deployment to `5`.

`kubectl scale deployment/beebox-web --replicas=5`

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       beebox-web-5bdcc84cb9-lh7z6                1/1     Running   0          4s
default       beebox-web-5bdcc84cb9-n98pz                1/1     Running   0          3m12s
default       beebox-web-5bdcc84cb9-nlfm8                1/1     Running   0          4s
default       beebox-web-5bdcc84cb9-s95sr                1/1     Running   0          3m10s
default       beebox-web-5bdcc84cb9-tjt7w                1/1     Running   0          4s
```