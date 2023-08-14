# Kubernetes Networking

Defines how Pods communicate to one another, regardless of which Node they are running on. Various Container Network Interface (CNI) implementations exist (i.e. Calico).

Each Pod has its own unique IP address within the cluster.

The cluster maintains a Virtual Network across Nodes so that any Pod can reach any other Pod via the Pod IP address. 

## CNI Plugins

Many CNI network plugins are available. These plugins provide network connectivity implementations.

Examples:
 - Calico
 - Flannel
 - Cilium
 - WeaveNet
 - Canal



> A Node will remain **NotReady** (no Pods will run) until a network plugin is installed.

## Notes from Docs

- Containers within a Pod can all reach each other's ports on `localhost`.
- Host Ports: 

- System component pods use the host network: `hostNetwork: true`, and share the network interface with the host VM. While non-system components are deployed to the pod overlay network. This is why system components can be started when the node is NotReady.

- 


TO DO:
 - Can a Pod have a random hostPort provided that is referenced by the services calling it, or must hostPort be hard-coded?


## DNS

Allows Pods to locate other Pods and Services using domain names instead of IP addresses. DNS runs as a Service, typically in the `kube-system` namespace. Commonly: CoreDNS.

### Pod IP Address Domain Names

`pod-ip-address.namespace-name.pod.cluster.local`

> Example: `192-168-10-100.default.pod.cluster.local`

## Lab

You are working for a company called BeeBox, a subscription service that ships weekly shipments of bees to customers. The company is using Kubernetes to run their infrastructure of containerized applications.

The company has set up a new development cluster to be used by an external contractor's development team. The cluster seems to be working and the team is able to access it, but they are reporting there is an issue.

### Fix the Issue Causing Pods Not to Start Up

First, you will need to figure out why the pods are not starting up. Check the pod status as well as the node status. Once you figure out why the pods are not starting up, fix the problem.

```
cloud_user@k8s-control:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
default       cyberdyne-frontend                    0/1     Pending   0          61m
default       testclient                            0/1     Pending   0          61m
kube-system   coredns-5d78c9869d-b2qqq              0/1     Pending   0          61m
kube-system   coredns-5d78c9869d-vh2h6              0/1     Pending   0          61m
kube-system   etcd-k8s-control                      1/1     Running   0          61m
kube-system   kube-apiserver-k8s-control            1/1     Running   0          61m
kube-system   kube-controller-manager-k8s-control   1/1     Running   0          61m
kube-system   kube-proxy-2n5n8                      1/1     Running   0          61m
kube-system   kube-proxy-b4xgq                      1/1     Running   0          61m
kube-system   kube-scheduler-k8s-control            1/1     Running   0          61m
cloud_user@k8s-control:~$ kubectl get nodes
NAME          STATUS     ROLES           AGE   VERSION
k8s-control   NotReady   control-plane   62m   v1.27.0
k8s-worker1   NotReady   <none>          61m   v1.27.0
```

All Nodes in NotReady status. Likely due to no network plugin running.

`ls /etc/cni/net.d`

No CNI plugins running.

`kubectl apply -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml`

Installed Calico.

```
$ kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-control   Ready    control-plane   69m   v1.27.0
k8s-worker1   Ready    <none>          69m   v1.27.0

$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       cyberdyne-frontend                         1/1     Running   0          70m
default       testclient                                 1/1     Running   0          70m
kube-system   calico-kube-controllers-7f48768458-w6v4z   1/1     Running   0          101s
kube-system   calico-node-6dvmh                          1/1     Running   0          102s
kube-system   calico-node-dc82j                          1/1     Running   0          102s
...
```

### Verify You Can Communicate between Pods Using the Cluster Network

To verify network connectivity, run a command in the testclient Pod to send an HTTP request to the cyberdyne-frontend on port 80.

```
$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
cyberdyne-frontend   1/1     Running   0          71m   192.168.194.65   k8s-worker1   <none>           <none>
testclient           1/1     Running   0          71m   192.168.194.66   k8s-worker1   <none>           <none>
```

`kubectl exec testclient -- curl 192.168.194.65`
`kubectl exec testclient -- curl 192-168-194-65.default.pod.cluster.local`

Received nginx welcome page. :thumbsup:

## Network Policies

An object that allows you to control the flow of network communication to and from Pods.

> By default, Pods are non-isolated and completely open to all communication. If any NetworkPoilcy selects a Pod, the Pod is considered isolated and will only be open to traffic allowed by NetworkPolicies.

Configs:

 - podSelector: determines which Pods in the namespace the NetworkPolicy applies. 
 - namespaceSelector: allow traffic from/to a namespace
 - ipBlock: a CIDR block to apply to
 - ports: specifies one or more port that will allow traffic. Traffic only allowed if it matches the `port` and one of from/to rules.
 - policyTypes: can apply to:
    - Ingress Traffic: selected by a `from` selector
    - Egress Traffic: selected by a `to` selector
    - Both

