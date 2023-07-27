Kubernetes Management
=================================================

High Availability
-----------------

How to achieve HA:
 - Multiple control plane nodes
   - kube-api-server runs on each control plane node
   - Load balancer balances kubectl and kubelet traffic to each of the kube-api-server instances
 - etcd HA
   - Stacked etcd - Instance of `etcd` runs on each of the control plane nodes
   - External etcd - `etcd` runs externally of cluster, on a HA cluster of etcd.

K8s Management Tools
--------------------

Tools
 - `kubectl` - CLI interface for k8s
 - `kubeadm` - quickly and easily create k8s clusters, set up contorl plane and working nodes
 - Minikube - for setting up single-node cluster for dev
 - Helm - templating and package management for k8s objects
 - Kompose - way of transitioning from Docker Compose to k8s components
 - Kustomize - config management tool for k8s objects

Safely Draining a K8s Node
--------------------------

Draining
 - Remove k8s node from service for performining maintenance
 - Move services to different nodes

#### Drain the node:

`kubectl drain <node name> --ignore-daemonsets`

> if you have daemonsets, and don't include the ignore flag, will get an error

Will set the node `Status=SchedulingDisabled`

#### Uncordoning the node:

`kubectl uncordon <node name>`

> note: won't automatically rebalance pods

#### Pods vs. Deployments:
 - drain cannot delete pods not managed by Deployment, ReplicaSet, Job, StatefulSet etc. - must use `--force` option and pod will be deleted
 - drain cannot delete DaemonSet pods - must use `--ignore-daemonsets`


Upgrading with kubeadm
----------------------

### On First Control Plane Node

1. Drain the control plane node

   `kubectl drain k8s-control --ignore-daemonsets`

2. Upgrade kubeadm on control plane node

   ```
   sudo apt-get update && \
   sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00
   ```

   > `--allow-change-held-packages` is because this package was held for auto upgrade during install

   > Find latest patch release using OS package manager:
   > ```
   > apt update
   > apt-cache madison kubeadm
   > ```

3. Plan the upgrade

   `sudo kubeadm upgrade plan v1.27.2`

   > Creates a plan of internal components that will be updated
   > Also outputs the required command to apply upgrade

4. Apply the upgrade

   `sudo kubeadm upgrade apply v1.27.2`

5. Upgrade kubelet and kubectl on the control plane node

   ```
   sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00

   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   sudo systemctl status kubelet
   ```

6. Uncordon the control plane node

   `kubectl uncordon k8s-control`

7. Confirm status and version

   `kubectl get nodes`

   Status = Ready
   Version = 1.27.2

### On Additional Control Plane Nodes, then Working Nodes

1. Drain worker node

   `kubectl drain k8s-worker1 --ignore-daemonsets --force`

2. Upgrade kubeadm

   ```
   sudo apt-get update && \
   sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00
   ```

3. Upgrade the kubelet config

   `sudo kubeadm upgrade node`

   > Note: do not need to run plan/apply or specify version - this command fetches the `ClusterConfiguration` from the cluster and upgrades based on that.

4. Upgrade kubelet and kubectl

   ```
   sudo apt-get update && \
   sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00

   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

5. Uncordon worker node

   `kubectl uncordon k8s-worker1`

6. Confirm status/version

   `kubectl get nodes`

7. Repeat on next node

Lab Notes
---------



Docs
----

 - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/ 