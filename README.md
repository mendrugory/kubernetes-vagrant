# Kubernetes-Vagrant

This a project which could be used it as playground to learn and experiment with [Kubernetes](https://kubernetes.io/), especially for **on-premise** installations.

> The code is just for fun and learning, do not use for production.
> Minimum 8 GB of RAM but 16 GB of RAM are recommended.

## Virtual Infrastructure

[Vagrant](https://www.vagrantup.com/) with [Virtualbox](https://www.virtualbox.org/) as provider will be used to create the infrastructure. One server will the Kubernetes master and two servers as Kubernetes workers. The virtual machines will run Ubuntu 16.04 (xenial).

Start up the infrastructure:

```
$ vagrant up
```

If any change is desired, you only have to modify [Vagrantfile](https://github.com/mendrugory/kubernetes-vagrant/blob/master/Vagrantfile).


## Kubernetes Installation

[Ansible](https://www.ansible.com/) is the chosen tool to install all the necessary software in the servers to have Kubernetes ready.

Several Roles have been developed to help us:

* docker: It will install docker in all the servers
* kubernetes-common: Common parts for Kubernetes installation which have to be installed in all the servers (masters and workers).
* kubernetes-master: It will initialize Kubernetes master, install a CNI plugin (Flannel in this case), or produce the code for the nodes to join the cluster.
* kubernetes-worker: It will help the nodes to join the cluster.

## Pointing to the Kubernetes cluster

The Ansible Playbook finishes with a task which will create a file called `mykubeconfig` which should be used to work with the just created kubernetes cluster through the tool `kubectl`

Example: 

```
$ kubectl get nodes -o wide --kubeconfig mykubeconfig
NAME   STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s1   Ready    master   1m      v1.13.5   192.168.33.11   <none>        Ubuntu 16.04.6 LTS   4.4.0-151-generic   docker://18.6.1
k8s2   Ready    <none>   57s     v1.13.5   192.168.33.21   <none>        Ubuntu 16.04.6 LTS   4.4.0-151-generic   docker://18.6.1
k8s3   Ready    <none>   57s     v1.13.5   192.168.33.22   <none>        Ubuntu 16.04.6 LTS   4.4.0-151-generic   docker://18.6.1
``` 


## Deploying Applications

### Deploy 2 replicas of Nginx and expose them.

```
$ kubectl run nginx --port=80 --image=nginx --replicas=2 --kubeconfig mykubeconfig 
$ kubectl expose deployment nginx --port=80 --type=NodePort --kubeconfig mykubeconfig 
```


### Deploy 2 replicas of Apache Web server and expose them.
```
$ kubectl run httpd --port=80 --image=httpd --replicas=2 --kubeconfig mykubeconfig 
$ kubectl expose deployment httpd --port=80 --type=NodePort --kubeconfig mykubeconfig 
```


### Visit the Applications

Check out the assigned Ports.

```
kubectl get svc --kubeconfig mykubeconfig 
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
httpd        NodePort    10.103.208.92   <none>        80:30275/TCP   18s
nginx        NodePort    10.105.235.99   <none>        80:32741/TCP   70s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        8m
```


Make requests to any of the nodes

```
$ curl 192.168.33.21:32741
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
$ curl 192.168.33.21:30275
<html><body><h1>It works!</h1></body></html>
```


## Exposing Applications

The servers only have private IPs, therefore it would not be possible to access to the applications from other computers. Let's solve it !!

* [Load Balancer](https://github.com/mendrugory/kubernetes-vagrant/tree/master/lb)
* [Ingress](https://github.com/mendrugory/kubernetes-vagrant/tree/master/ingress)