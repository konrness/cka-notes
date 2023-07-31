kubectl
=================================================

kubectl get
-----------------

List objects in cluster

`kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>`

 - `<object type>` must be one from list of command `kubectl api-resources`
 - `-o` - set output format
 - `--sort-by` - sort output using JSONPath expression
 - `--selector` - filter results by label

Examples:
 - `kubectl get pods my-pod`
 - `kubectl get pods -o wide`
 - `kubectl get pods -o json`
 - `kubectl get pods -o yaml`
 - `kubectl get pods -o wide --sort-by .spec.nodeName`
 - `kubectl get pods -n kube-system`
 - `kubectl get pods -n kube-system --selector k8s-app=calico-node`

kubectl describe
----------------

Detailed information about a specific object

Examples:
 - `kubectl describe pod my-pod`

kubectl create
--------------

`kubectl create` - Create objects imperitively.

> Will return an error if the object already exists

`kubectl create -f` - Create an object from YAML file descriptor

kubectl apply
-------------

Similar to create, but will allow to modify an existing object.

kubectl delete
--------------

Delete object

Example:
 - `kubectl delete pod my-pod`

kubectl exec
------------

Run commands inside containers.
`kubectl exec <podname> -c <containername> -- <command>`

 - `-c <containername>` only required if pod has multiple containers

Example:
 - `kubectl exec my-pod -c busybox -- echo "Hello, world!"`

## Tips

### Declarative vs. Imperative

**Declarative** - Define objects using data structures such as YAML or JSON

**Imperative** - Define objects using kubectl commands and flags.

### Imperatively create declarative YAML

`kubectl create deployment my-deployment --image=nginx --dry-run -o yaml > my-deployment.yaml`

### Recording imperitive changes to objects

`kubectl scale deployment my-deployment --replicas 5 --record`

> `--record` flag used to record change-cause.

`kubectl describe deployment my-deployment`

> `Annotations: kubernetes.io/change-cause: ...`

## Lab 1

### Get a List of Persistent Volumes Sorted by Capacity

Use kubectl to get a list of persistent volumes. Ensure this list is sorted by the persistent volume capacity. Store the sorted kubectl output in the file /home/cloud_user/pv_list.txt.

1. `kubectl api-resources | grep persistent`, found persistent volumes object name is `persistentvolumes` or `pv`
2. `kubectl get pv`
3. `kubectl get pv pv0002 -o yaml` - found capacity value is at `.spec.capacity.storage`
4. `kubectl get pv --sort-by .spec.capacity.storage`

```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv0002       1Gi        RWO            Retain           Available           manual                  134m
audit-logs   2Gi        RWO            Retain           Available           manual                  134m
pv0003       3Gi        RWO            Retain           Available           manual                  134m
```

5. `kubectl get pv --sort-by .spec.capacity.storage > /home/cloud_user/pv_list.txt`

### Run a Command Inside the `quark` Pod's Container to Obtain a Key Value

Locate the quark pod within the beebox-mobile namespace. Run a command inside this pod's container to obtain a key value. The key value is located inside the container's file system in a file called /etc/key/key.txt. Save the key value to a file on the Kube control plane server at /home/cloud_user/key.txt.

1. `kubectl get pods -n beebox-mobile`
2. `kubectl exec quark -n beebox-mobile -- cat /etc/key/key.txt > key.txt`

### Create a Deployment Using a Spec File

You will find a deployment spec file located at /home/cloud_user/deployment.yml. Using this file, create the deployment it describes in the cluster.

1. `kubectl apply -f deployment.yml`
2. `kubectl describe deployment nginx-deployment -n beebox-mobile`

### Delete the `beebox-auth` Service

There is a service in the beebox-mobile namespace called beebox-auth-svc. Delete this service.

1. `kubectl get services -n beebox-mobile`
2. `kubectl delete service beebox-auth-svc -n beebox-mobile`

## Lab 2

