Kubernetes Role Based Access Control
=================================================

RBAC Objects
-----------------

### Roles and ClusterRoles

Define a set of permissions.

Roles define permission with a particular namespace, and a ClusterRole defines cluster-wide permissions not specific to a namespace.

### RoleBinding and ClusterRoleBinding

Connects Roles and ClusterRoles to users.

# Lab

Your developers frequently request that you provide information from the Kubernetes cluster, so you would like to give them the ability to read data from the cluster but not make any changes to it. Using Kubernetes role-based access control, ensure the `dev` user can read pod metadata and container logs from any pod in the `beebox-mobile` namespace.

A kubeconfig file for the `dev` user has already been created on the server. You can use this file to test your RBAC setup as the `dev` user like so:

`kubectl get pods -n beebox-mobile --kubeconfig /home/cloud_user/dev-k8s-config`

## Create a Role for the `dev` User

Create a role called pod-reader. Provide it with read access to pods and container logs in the beebox-mobile namespace.

```
$ kubectl get pods -n beebox-mobile --kubeconfig dev-k8s-config
Error from server (Forbidden): pods is forbidden: User "dev" cannot list resource "pods" in API group "" in the namespace "beebox-mobile"
```

1. `kubectl create role pod-reader -n beebox-mobile --resource=pod,pod/log --verb=read,watch,list --dry-run -o yaml > pod-reader.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: beebox-mobile
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  verbs:
  - get
  - watch
  - list
```

2. `kubectl apply -f pod-reader.yaml`


## Bind the Role to the `dev` User and Verify Your Setup Works

Create a RoleBinding to bind the pod-reader role to the dev user. Interact with pods in the beebox-mobile namespace to make sure you can read pod metadata and container logs as the dev user but not make any changes.

1. `kubectl create rolebinding dev-pod-reader --role=pod-reader --user=dev -n beebox-mobile --dry-run -o yaml > dev-pod-reader.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: dev-pod-reader
  namespace: beebox-mobile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev
```

2. `kubectl apply -f dev-pod-reader.yaml`
3. `kubectl get pods -n beebox-mobile --kubeconfig dev-k8s-config`

```
NAME          READY   STATUS    RESTARTS   AGE
beebox-auth   1/1     Running   0          108m
```

```
cloud_user@k8s-control:~$ kubectl logs beebox-auth -n beebox-mobile --kubeconfig dev-k8s-config
"Auth processing..."
"Auth processing..."
cloud_user@k8s-control:~$ kubectl delete pod beebox-auth -n beebox-mobile --kubeconfig dev-k8s-config
Error from server (Forbidden): pods "beebox-auth" is forbidden: User "dev" cannot delete resource "pods" in API group "" in the namespace "beebox-mobile"
```