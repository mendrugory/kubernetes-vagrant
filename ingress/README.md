# Ingress

Exposing traffic using load balancers have a drawback and complete load balancer is used for only one service-application. Ingresses try to help to serve more than one service. One of the most used is [Nginx](https://www.nginx.com/).

We are going to try to expose two applications (nginx web server and apache web server) through our own ingress. An Ansible role will deploy a Nginx which will route the traffic based on the prefix url:

* /app1 => to nginx
* /app2 => to apache


The virtual machine will receive a public IP which will be accessible by other computers in the same network.

## Start up the Ingress

### Start up the virtual machine

```
$ vagrant up
```

### Set up the Ingress

It is necessary to pass the IPs of the nodes and the ports of the apps.

```
$ ansible-playbook ingress.yml --extra-vars "server1=192.168.33.21 server2=192.168.33.22 app1_port=32741 app2_port=30275"
```


> Check the given Public IP.

### Visit the applications through the Ingress

```
$ curl 192.168.1.144/app1/
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
$ curl 192.168.1.144/app2/
<html><body><h1>It works!</h1></body></html>
```