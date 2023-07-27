Building a Kubernetes 1.27 Cluster with `kubeadm`
=================================================

Introduction
------------

This lab will allow you to practice the process of building a new Kubernetes cluster. You will be given a set of Linux servers, and you will have the opportunity to turn these servers into a functioning Kubernetes cluster. This will help you build the skills necessary to create your own Kubernetes clusters in the real world.

Scenario
--------

Your company wants to transition to a Kubernetes-based infrastructure for running their containerized applications. In order to do that, they will need a basic Kubernetes cluster to get started.

You have been provided with three servers. Build a simple Kubernetes cluster with one control plane node and two worker nodes.

 - Install and use `kubeadm` to build a Kubernetes cluster on these servers.
 - Install Kubernetes version `1.27.0`.
 - Use `containerd` for your container runtime.
 - The cluster should have one control plane node and two worker nodes.
 - Use the Calico networking add-on to provide networking for the cluster.


Solution
--------

### Set Hostnames of Control and Worker nodes

1. Set hostnames

    `sudo hostnamectl set-hostname [k8s-control | k8s-worker1 | k8s-worker2]`

2. Populate hosts file

    `sudo vi /etc/hosts`

    > Place the entries for each of the nodes of the cluster in each node's hosts file. The hosts file should contain the private IP of each node along with its corresponding hostname.

    Example:

    ```
    10.0.1.101 k8s-control
    10.0.1.102 k8s-worker1
    10.0.1.103 k8s-worker2
    ```

### Install Packages

1.  Log in to the control plane node.
    
    > **Note:** The following steps must be performed on all three nodes.
    
2.  Create the configuration file for containerd:
    
    `cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf overlay br_netfilter EOF`
    
3.  Load the modules:
    
    `sudo modprobe overlay sudo modprobe br_netfilter`
    
4.  Set the system configurations for Kubernetes networking:
    
    `cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf net.bridge.bridge-nf-call-iptables = 1 net.ipv4.ip_forward = 1 net.bridge.bridge-nf-call-ip6tables = 1 EOF`
    
5.  Apply the new settings:
    
    `sudo sysctl --system`
    
6.  Install containerd:
    
    `sudo apt-get update && sudo apt-get install -y containerd.io`
    
7.  Create the default configuration file for containerd:
    
    `sudo mkdir -p /etc/containerd`
    
8.  Generate the default containerd configuration, and save it to the newly created default file:
    
    `sudo containerd config default | sudo tee /etc/containerd/config.toml`
    
9.  Restart containerd to ensure the new configuration file is used:
    
    `sudo systemctl restart containerd`
    
10.  Verify that containerd is running:
    
    `sudo systemctl status containerd`
    
11.  Disable swap:
    
    `sudo swapoff -a`
    
12.  Install the dependency packages:
    
    `sudo apt-get update && sudo apt-get install -y apt-transport-https curl`
    
13.  Download and add the GPG key:
    
    `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`
    
14.  Add Kubernetes to the repository list:
    
    `cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list deb https://apt.kubernetes.io/ kubernetes-xenial main EOF`
    
15.  Update the package listings:
    
    `sudo apt-get update`
    
16.  Install Kubernetes packages:
    
    > **Note:** If you get a `dpkg lock` message, just wait a minute or two before trying the command again.
    
    `sudo apt-get install -y kubelet=1.27.0-00 kubeadm=1.27.0-00 kubectl=1.27.0-00`
    
17.  Turn off automatic updates:
    
    `sudo apt-mark hold kubelet kubeadm kubectl`
    
18.  Log in to both worker nodes to perform the previous steps.
    

### Initialize the Cluster

1.  Initialize the Kubernetes cluster on the control plane node using `kubeadm`:
    
    `sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.0`
    
2.  Set `kubectl` access:
    
    `mkdir -p $HOME/.kube sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOME/.kube/config`
    
3.  Test access to the cluster:
    
    `kubectl get nodes`
    

### Install the Calico Network Add-On

1.  On the control plane node, install Calico Networking:
    
    `kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml`
    
2.  Check the status of the control plane node:
    
    `kubectl get nodes`
    

### Join the Worker Nodes to the Cluster

1.  In the control plane node, create the token and copy the `kubeadm join` command:
    
    `kubeadm token create --print-join-command`
    
    > **Note:** This output will be used as the next command for the worker nodes.
    
2.  Copy the full output from the previous command used in the control plane node. This command starts with `kubeadm join`.
    
3.  In both worker nodes, paste the full `kubeadm join` command to join the cluster. Use `sudo` to run it as root:
    
    `sudo kubeadm join...`
    
4.  In the control plane node, view the cluster status:
    
    `kubectl get nodes`
    
    > **Note:** You may have to wait a few moments to allow all nodes to become ready.
    

Conclusion
----------

Congratulations — you've completed this hands-on lab!