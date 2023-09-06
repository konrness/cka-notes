# Lab - Part 1

This question uses the `acgk8s` cluster. After logging in to the exam server, switch to the correct context with the command `kubectl config use-context acgk8s`.

Each of the objectives represents a task which you will need to complete using the available cluster and server(s). Read each objective carefully and complete the task specified.

For some objectives, you may need to ssh into other nodes or servers from the exam server. You can do so using the hostname/node name ( i.e., ssh acgk8s-worker1).

Note: You cannot ssh into another node, or use kubectl to connect to the cluster, from any node other than the root node. Once you have completed the necessary tasks on a server, be sure to exit and return to the root node before proceeding.

If you need to assume root privileges on a server, you can do so with sudo -i.

You can run the verification script located at /home/cloud_user/verify.sh at any time to check your work!

### Count the Number of Nodes That Are Ready to Run Normal Workloads

Determine how many nodes in the cluster are ready to run normal workloads (i.e., workloads that do not have any special tolerations).

Output this number to the file /k8s/0001/count.txt.

```
$ kubectl get nodes
NAME             STATUS   ROLES           AGE     VERSION
acgk8s-control   Ready    control-plane   5m47s   v1.27.0
acgk8s-worker1   Ready    <none>          5m7s    v1.27.0
acgk8s-worker2   Ready    <none>          5m6s    v1.27.0

echo "2" >> /k8s/0001/count.txt
```

### Retrieve Error Messages from a Container Log

In the backend namespace, check the log for the proc container in the data-handler Pod.

Save the lines which contain the text ERROR to the file /k8s/0002/errors.txt.

```
$ kubectl get pods -n backend -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
data-handler   1/1     Running   0          6m36s   192.168.19.1   acgk8s-worker2   <none>           <none>

$ kubectl logs -n backend data-handler -c proc | grep ERROR
[ERROR] Could not process record 006c27
[ERROR] Could not process record 01a45c
cloud_user@acgk8s-control:~$ kubectl logs -n backend data-handler -c proc | grep ERROR > /k8s/0002/errors.txt
```

### Find the Pod with a Label of app=auth in the Web Namespace That Is Utilizing the Most CPU

Before doing this step, please wait a minute or two to give our backend script time to generate CPU load. Determine which Pod in the web namespace with the label app=auth is using the most CPU. Save the name of this Pod to the file /k8s/0003/cpu-pod.txt.

```
$ kubectl top pod -n web --selector="app=auth" --sort-by=cpu
NAME       CPU(cores)   MEMORY(bytes)
auth-web   100m         6Mi
auth1      0m           0Mi
auth2      0m           0Mi

echo "auth-web" > /k8s/0003/cpu-pod.txt
```

