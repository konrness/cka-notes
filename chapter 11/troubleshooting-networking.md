# Troubleshooting Networking

## kube-proxy and DNS

Important.

`kubectl get pods -n kube-system`

`kubectl logs -n kube-system kube-proxy-[id]`

`kubectl logs -n kube-system kube-dns-[id]`

## netshoot

Run a container in the cluster to run commands and test network functionality.

Image: `nicolaka/netshoot`

Includes:
 - curl
 - ping
 - nslookup
 - etc...

