# Load Balancer

When you run a Kubernetes cluster offered by a Cloud provider and you wish to expose your services using a Load Balancer Service, a load balancer which belongs to the cloud provider (for instance [AWS ELB](https://aws.amazon.com/elasticloadbalancing/)) will be launched and it will serve our application to the outside world.

We will try to replicate that behaviour. A new virtual machine will be launched (check [Vagrantfile](Vagrantfile)) as load balancer. Instead of installing any software product with the capacity of balancing requests between our replicas, we will use [Netfilter](https://en.wikipedia.org/wiki/Netfilter) through the tool `iptables`. The configuration will be applied by the Ansible role `load-balancer`.

The virtual machine will receive a public IP which will be accessible by other computers in the same network.

## Start up the Load Balancer

### Start up the virtual machine

```
vagrant up
```

### Set up the load balancer for the Nginx

It is necessary to pass the IPs of the nodes, the port of the app and the network interface (by default will be `enp0s9`).

```
ansible-playbook lb.yml --extra-vars "server1=192.168.33.21 server2=192.168.33.22 app_port=31603 interface=enp0s9"
```

> Check the given Public IP.

### Visit the application through the Load Balancer

```
curl 192.168.1.143
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
