# Local K8s Cluster

## Credits

Simple set up ansible scripts based on a [DO](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04) tutorial.

## Deviation

While generally the same, I had updated for Ubuntu 20.04 (minimal changes), and use my own username. The largest changes were:

* that I am using the latest version(s) of the tools vs pinned at v1.14
* an updated flannel URL for later versions.
* username change
* dropped the use of `ansible_user` in the *hosts* file.

## Use

* deploy 4 Ubuntu VM's (or hosts)
* ***All commands are on your local host unless otherwise specified***.
* clone this repo
* create a `hosts` file (root of this repo) in the format of:

```ini
[masters]
master ansible_host=master1_ip

[workers]
worker1 ansible_host=worker1_ip
worker2 ansible_host=worker2_ip
worker3 ansible_host=worker3_ip

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

* create your user and add ssh keys. `ansible-playbook -i hosts init.yml`
* install Docker, k8s tools and deps. `ansible-playbook -i hosts k8s-deps.yml`
* [*OPTIONAL*] You can edit `--pod-network-cidr=10.244.0.0/16` in the `master.yml` if you prefer a different pod network.
* Init the master/control-plane, setup Flannel, setup `admin.conf` (on master), etc. `ansible-playbook -i hosts master.yml`
* ssh into the master (From Host: `ssh user@master_ip`) and check that it is ready (On master node: `kubectl get nodes`). Wait until you see the node is ready with something like:

```shell
$ kubectl get nodes
NAME   STATUS   ROLES                  AGE   VERSION
k8c    Ready    control-plane,master   56m   v1.20.2
```

* If all good, you can exit the ssh session back to your local host and continue.
* Add the worker nodes to the cluster. `ansible-playbook -i hosts workers.yml`
* Verify the cluster - ssh into the master (From Host: `ssh user@master_ip`) and check that it is ready (On master node: `kubectl get nodes`). Wait until you see the nodes are ready with something like:

```shell
$ kubectl get nodes
NAME   STATUS   ROLES                  AGE   VERSION
k81    Ready    <none>                 17m   v1.20.2
k82    Ready    <none>                 17m   v1.20.2
k83    Ready    <none>                 17m   v1.20.2
k8c    Ready    control-plane,master   56m   v1.20.2
```

* If all of your nodes have the value Ready for STATUS, it means that they’re part of the cluster and ready to run workloads.
* You are now ready to deploy a workload via `kubectl` or similar tools.
