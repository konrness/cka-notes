# Storage

## Container File Systems

Pod file system is ephemeral. If containers are deleted or re-created, the file system is lost.

## Volumes

Persistent method of data storage. Stored outside of the container file system allowin containers to access at runtime.

Volume is defined in the Pod Spec. Volume Mounts are defined in container spec, and define how volumes are mounted into each container.

> You can mount the same volume to multiple containers in the same pod. Common for sidecar containers that process/transform files.

Common Volume Types:
 - hostPath - stored in a specific directory on node
 - emptyDir - temp volume, exists only as long as the Pod. Most common for sharing data between two containers in a pod.

## PersistentVolumes

Define storage as an abstract resource to be consumed by pods.

Storage Classes: allow you to define different types of storage in an environment.

> `persistentVolumeReclaimPolicy`: if not set to `true`, resizing will result in an error.

reclaimPolicies: determines how storage resources can be reused, when PVCs are deleted.
 - Retain - keep all data, requires manual cleanup
 - Delete - only works for cloud storage resources, automatically delete persistent volume and cloud resources.
 - Recycle - automatically deletes data (`rm -rf`), allows PV to be reused after recycling.

Volume Types
 - NFS
 - Cloud Storage
 - ConfigMaps and Secrets
 - Simple Directory on Node

### PersistentVolumeClaims

Represents a request for storage resources. Defines attributes, like PV (e.g. StorageClass, etc.)

When PVC is created, K8s looks for a PV that is able to satisfy the requested criteria. If one is found, it will be bound to the PV.

PVCs can be expanded without interrupting apps using them. Just edit the `spec.resources.requests.storage` of PVC. Only works if `allowVolumeExpansion=true`.

## Lab 1 - Volumes

Your company, BeeBox, is developing some applications that have storage needs beyond the short-lived storage of the container file system. One component, a simple maintenance script, needs to be able to interact with a directory on the host file system. Another needs to be able to share data between two containers in the same Pod.

Your task is to build a simple architecture to demonstrate how these tasks can be accomplished with Kubernetes volumes. Create Pods that meet the specified criteria.

### Create a Pod That Outputs Data to the Host Using a Volume

One of the applications under development will be a maintenance script that needs to write data to the host's file system.

Create a Pod that outputs some data every 5 seconds to the host's disk in the directory `/var/data`.


See [host-logs-pod.yaml](host-logs-pod.yaml).

### Create a Multi-Container Pod That Shares Data Between Containers Using a Volume

Another application component includes two pieces of software that need to collaborate using shared data.

Create a multi-container Pod with a volume mounted to both containers.

See [multi-container.yaml](multi-container.yaml).

## Lab 2 - Persistent Volumes

Your company, BeeBox, is developing some applications for Kubernetes. They are anticipating some increasingly complex storage needs in the future, so they want to make sure that they can leverage the full potential of Kubernetes storage with PersistentVolumes.

Your task is to build an application that uses a PersistentVolume for storage. Ensure that the application's volume is able to be expanded in case its storage needs might increase later.

### Create a PersistentVolume That Allows Claim Expansion

Create a hostPath PersistentVolume with a size of 1Gi that stores data on the host at `/var/output`. Ensure that this PersistentVolume is configured so that Pod volumes that use it can be expanded later. In addition, configure the volume so that it will be automatically reusable if the claims using it are deleted.

Note that you may need to create other objects in addition to the PersistentVolume object.

See [pv-claim-expansion.yaml](pv-claim-expansion.yaml).

### Create a PersistentVolumeClaim

Create a PersistentVolumeClaim with a size of 100Mi. Make sure it is able to bind to the PersistentVolume.

### Create a Pod That Uses a PersistentVolume for Storage

Create a Pod that uses the PersistentVolume and writes some data to it.

