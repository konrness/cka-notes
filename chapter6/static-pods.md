# Static Pods

A Pod that is managed directly by teh kubelet on a node, not the k8s API server. They can run even if there is no API server present.

Automatically created from YAML manifest files located in the manifest path on the node.

Default Path: `/etc/kubernetes/manifests/`

Changes are applied when kubelet is restarted:

`sudo systemctl restart kubelet`

## Mirror Pods

Kubelet will create a mirror Pod for each static Pod, to see the status of the static pod via k8s API. You cannot make changes to the mirror pod.


## Lab

You are working for BeeBox, a company that provides regular shipments of bees to customers.

The company has built a special diagnostic tool for its K8s nodes. This tool can be run as a container in a K8s Pod, and it collects detailed diagnostic data from the worker node throughout the node lifecycle.

One particularly useful feature is that this tool is able to collect data during the node startup process, before the kubelet begins communicating with the Kubernetes API, or even before a kubelet joins a cluster. To benefit from this, the Pod needs to run without depending on the presence of a Kubernetes API server connection. It will need to be run and managed directly by the kubelet.

Your task is to create a pod to run this diagnostic tool on the `Worker Node 1` server. Use the `acgorg/beebox-diagnostic:1` image for this Pod.


```
apiVersion: v1
kind: Pod
metadata:
  name: beebox-diagnostic
spec:
  containers:
  - name: beebox-diagnostic
    image: acgorg/beebox-diagnostic:1

```