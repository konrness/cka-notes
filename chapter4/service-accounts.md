Service Accounts
=================================================

Purpose of Service Accounts
---------------------------

Account used by container processes within pods to authenticate with the k8s API. Pods that need to communicate with the k8s API use service accounts to control their access.

Usage
-----

### Create Service Accounts

Declarative YAML:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: default
```

Imperitively:

`kubectl create sa my-serviceaccount2 -n default`

### Binding Roles to Service Accounts

Bind using a `RoleBinding` or `ClusterRoleBinding` just like users.

```
...
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default

```

`kubectl describe sa my-serviceaccount -n default`