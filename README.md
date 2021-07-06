# Kubernetes-Vagrant

This a project which could be used it as playground to learn and experiment with [Kubernetes](https://kubernetes.io/), especially for **on-premise** installations.

> The code is just for fun and learning, do not use for production.
> Minimum 8 GB of RAM but 16 GB of RAM are recommended.

## Virtual Infrastructure

[Vagrant](https://www.vagrantup.com/) with [Virtualbox](https://www.virtualbox.org/) as provider will be used to create the infrastructure. One server will the Kubernetes master and two servers as Kubernetes workers. The virtual machines will run Ubuntu 20.04 (focal).

Start up the infrastructure:

```
vagrant up
```

If any change is desired, you only have to modify [Vagrantfile](https://github.com/mendrugory/kubernetes-vagrant/blob/master/Vagrantfile).


## Kubernetes Installation

[Ansible](https://www.ansible.com/) is the chosen tool to install all the necessary software in the servers to have Kubernetes ready.

Several Roles have been developed to help us:

* docker: It will install docker in all the servers
* kubernetes-common: Common parts for Kubernetes installation which have to be installed in all the servers (masters and workers).
* kubernetes-master: It will initialize Kubernetes master, install a CNI plugin (Flannel in this case), or produce the code for the nodes to join the cluster.
* kubernetes-worker: It will help the nodes to join the cluster.

### Installation

```bash
ansible-playbook k8s.yml
```


## Pointing to the Kubernetes cluster

The Ansible Playbook finishes with a task which will create a file called `mykubeconfig` which should be used to work with the just created kubernetes cluster through the tool `kubectl`

Example:

```
kubectl get nodes -o wide --kubeconfig mykubeconfig
```

```
NAME   STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
m01    Ready    master   2m36s   v1.18.0   192.168.33.11   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
w01    Ready    <none>   2m6s    v1.18.0   192.168.33.21   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
w02    Ready    <none>   2m6s    v1.18.0   192.168.33.22   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
```

You can also set the `KUBECONFIG` environment variable to path of `mykubeconfig` like this:
```
export KUBECONFIG=./mykubeconfig
```

And then run:
```
kubectl get nodes -o wide
```


## Deploying Applications

### Deploy 2 replicas of Nginx and expose them.

```
kubectl create deployment mynginx --image=nginx:1.15-alpine
```
```
kubectl scale deployment mynginx --replicas=3
```
```
kubectl expose deployment mynginx --port=80 --type=NodePort --name=mynginx-service
```


### Deploy 2 replicas of Apache Web server and expose them.
```
kubectl create deployment myhttpd --port=80 --image=httpd
```
```
kubectl scale deployment myhttpd --replicas=3
```
```
kubectl expose deployment myhttpd --port=80 --type=NodePort --name=myhttpd-service
```


### Visit the Applications

Check out the assigned Ports.

```
kubectl get svc
```

```
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        3h34m
myhttpd-service   NodePort    10.102.106.131   <none>        80:32443/TCP   3m30s
mynginx-service   NodePort    10.103.115.154   <none>        80:31603/TCP   10m
```


Make requests to any of the nodes

```
curl 192.168.33.21:31603
```

```
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

```
curl 192.168.33.21:32443
```
```
<html><body><h1>It works!</h1></body></html>
```


## Exposing Applications

The servers only have private IPs, therefore it would not be possible to access to the applications from other computers. Let's solve it !!

* [Load Balancer](lb)
* [Ingress](ingress)
