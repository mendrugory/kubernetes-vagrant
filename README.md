

```
$ vagrant up
```

```
$ kubectl run nginx --port=80 --image=nginx --replicas=2
$ kubectl expose deployment nginx --port=80 --type=NodePort
```
```
$ kubectl run httpd --port=80 --image=httpd --replicas=2
$ kubectl expose deployment httpd --port=80 --type=NodePort
```
```

```
$ kubectl get svc
```




sudo iptables -A FORWARD -o flannel.1 -j ACCEPT

