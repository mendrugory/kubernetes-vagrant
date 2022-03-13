# Kubernetes-Vagrant

This a project which could be used it as playground to learn and experiment with [Kubernetes](https://kubernetes.io/), especially for **on-premise** installations.

> The code is just for fun and learning, do not use for production.
> Minimum 8 GB of RAM but 16 GB of RAM are recommended.

## Prerequisites
- Vagrant
- Virtualbox
- Python3.8 or later (to run Ansible)

## Virtual Infrastructure

[Vagrant](https://www.vagrantup.com/) with [Virtualbox](https://www.virtualbox.org/) as provider will be used to create the infrastructure. One server will the Kubernetes master and two servers as Kubernetes workers. The virtual machines will run Ubuntu 20.04 (focal).

Start up the infrastructure:

```shell
vagrant up
```

## Install Ansible
Install virtual python environment:
```shell
python3.8 -m venv .venv
```

Activate the virtual environment:
```shell
source .venv/bin/activate
```

Install Ansible:
```shell
pip install ansible
```

## Kubernetes Installation

Ansible roles to setup the cluster:

* docker: It will install docker in all the servers
* kubernetes-common: Common parts for Kubernetes installation which have to be installed in all the servers (masters and workers).
* kubernetes-master: It will initialize Kubernetes master, install a CNI plugin (Flannel in this case), or produce the code for the nodes to join the cluster.
* kubernetes-worker: It will help the nodes to join the cluster.

### Installation
```shell
ansible-playbook k8s.yml
```

#### List the nodes

When Ansible Playbook finishes, a file called `mykubeconfig` is created. We use this file to communicate with the cluster.

Example:

```shell
kubectl --kubeconfig mykubeconfig get nodes -o wide
```

```
NAME   STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
m01    Ready    master   2m36s   v1.18.0   192.168.33.11   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
w01    Ready    <none>   2m6s    v1.18.0   192.168.33.21   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
w02    Ready    <none>   2m6s    v1.18.0   192.168.33.22   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.7
```

You can also set the `KUBECONFIG` environment variable to path of `mykubeconfig` like this:
```shell
export KUBECONFIG=./mykubeconfig
```

And then run:
```shell
kubectl get nodes -o wide
```

#### Optional Dashboard

Deploy the [Kubernetes Dashboard](https://github.com/kubernetes/dashboard):

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

Create a service account with admin role. ([source](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)):

```shell
kubectl apply -f jobs/dashboard/dashboard-service-account-with-admin-role.yaml
```

Copy the token from output of following command:

```shell
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

Make the dashboard accessible from your host (run this in a different command line session):

```shell
kubectl --kubeconfig mykubeconfig port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443
```

Navigate to https://localhost:8080/ and login with the token from above.

## Deploying Applications

### Deploy 2 replicas of Nginx and expose them

Build the special nginx image on the master1 and push it to its docker registry:
```shell
ansible master1 -b -m shell -a 'cd /vagrant/apps/nginx && docker build . -t localhost:5000/mynginx && docker push localhost:5000/mynginx'
```

Create a deployment:
```shell
kubectl create deployment mynginx --port=80 --image=192.168.33.11:5000/mynginx
```

Set number of required replicas:
```shell
kubectl scale deployment mynginx --replicas=3
```

Expose the service to a dynamically assigned port:
```shell
kubectl expose deployment mynginx --port=80 --type=NodePort --name=mynginx
```

### Visit the Applications

Check out the assigned Ports.

```shell
kubectl get svc
```

```
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        3h34m
mynginx-service   NodePort    10.103.115.154   <none>        80:31603/TCP   10m
```

You can now access _any_ of the worker nodes in the kubernetes cluster on the `assigned` port (31603 in our case) to access the application.

```shell
ASSIGNED_PORT=31603
for i in 21 22; do echo -n "192.168.33.$i: "; curl http://192.168.33.$i:$ASSIGNED_PORT; done;
```

You should see something like this:
```shell
192.168.33.21: Sunday, 27-Jun-2021 13:24:23 UTC: mynginx-76b9d5cb7b-jfrvk
192.168.33.22: Sunday, 27-Jun-2021 13:24:23 UTC: mynginx-76b9d5cb7b-8j8lg
```


## Exposing Applications

The servers only have private IPs, therefore it would not be possible to access to the applications from other computers. Let's solve it !!

* [Load Balancer](lb)
* [Ingress](ingress)
