# Application Configuration

### ConfigMaps

Key-Value Map of configuration data


### Secrets

for Passwords/API Keys

Values must be base-64 encoded

```
echo -n 'secret' | base64
```

### Environment Variables

Pass ConfigMap and Secret data to app as environment variable

### Configuration Volumes

Config data passed in the form of a Mounted Volume, as files on the filesystem.

One file for each top-level key of the configs.

Secrets are base64 decoded before file is written.


## Lab

### Scenario

One of these applications is a simple Nginx web server. This server is used as part of a secure backend application, and the company would like it to be configured to use HTTP basic authentication.

This will require an `htpasswd` file as well as a custom Nginx config file. In order to deploy this Nginx server to the cluster with good configuration practices, you will need to load the custom Nginx configuration from a ConfigMap (this already exists) and use a Secret to store the `htpasswd` data.

Create a Pod with a container running the `nginx:1.19.1` image. Supply a custom Nginx configuration using a ConfigMap, and populate an `htpasswd` file using a Secret.

`htpasswd` is already installed on the server, and you can generate an `htpasswd` file like so:

`htpasswd -c .htpasswd user`

A pod called `busybox` already exists in the cluster, which you can use to contact your Nginx pod and test your setup.

### ConfigMap



```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  nginx.conf: |
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
      worker_connections  1024;
    }
    http {
        server {
            listen       80;
            listen  [::]:80;
            server_name  localhost;
            location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
            }
            auth_basic "Secure Site";
            auth_basic_user_file conf/.htpasswd;
        }
    }
```

### htpasswd

`echo "test101" | htpasswd -ni user | base64`

> `dXNlcjokYXByMSRET0VwdlEwOCRsTGNIblpBNHVMRG5ON3o3UWNWMkgxCgo=`

```
apiVersion: v1
kind: Secret
metadata:
  name: nginx-htpasswd
data:
  .htpasswd: dXNlcjokYXByMSRET0VwdlEwOCRsTGNIblpBNHVMRG5ON3o3UWNWMkgxCgo=
```

Alternatively:

`kubectl create secret generic nginx-htpasswd --from-file .htpasswd`

### Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx
    - name: nginx-htpasswd
      mountPath: /etc/nginx/conf
  volumes:
  - name: nginx-config
    configMap:
      name: nginx-config
  - name: nginx-htpasswd
    secret:
      secretName: nginx-htpasswd
```

### Testing

```
$ kubectl get pods nginx-pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          12s   192.168.194.71   k8s-worker1   <none>           <none>

$ curl http://192.168.194.71:80
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.19.1</center>
</body>
</html>



$ curl http://192.168.194.71:80 -u "user:test101"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```