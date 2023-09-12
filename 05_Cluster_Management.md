## High Availability Control Plane
When using multiple control planes for HA, you will likely need to communicate with the K8s API through a **load balancer**. This includes clients such as `kubelet` instances running on worker nodes.
![[Pasted image 20230827203914.png]]
Another concept when understanding High Availability in the context of K8s, that concept deals with how we manage our `etcd` instances, there are multiple design patterns that you can use in the context of a **multi control plane high availability Kubernetes setup**, here are some:
### Stacked `etcd`
`etcd` runs in the same nodes as the rest of the control plane components and this is actually the design pattern that we have been using throughout the course because this is the design pattern that `kubeadm` follows.
![[Pasted image 20230827204619.png]]
### External `etcd`
`etcd` runs completely on separate nodes, so in the context of high availability we could have multiple external `etcd` nodes in a high availability cluster and that set of servers would be a completely different set of servers from the ones that are running our normal K8s control plane components. With this model, you can have any number of K8s control plane instances and any number of `etcd` nodes.
![[Pasted image 20230827205005.png]]
## K8s Management Tools
These tools interface with K8s to provide additional functionality.
### `kubectl`
This is the official CLI for K8s.
### `kubeadm`
A tool for quickly and easily creating K8s clusters.
### `minikube`
A tool for automatically setting up a local single-node K8s cluster. It's great for getting K8s up an running quickly for development purposes.
### `Helm`
It provides templating and package management for K8s objects, you can use it to manage your own templates (known as charts), you can also download and use templates.
### `Kompose`
Translate Docker compose files into K8s objects.
### `Kustomize`
A configuration management tool for managing Kubernetes object configurations, you can share and re-use templated configurations for K8s applications.
## Safely Draining A K8s Node
### What is draining?
Sometimes you need to remove a K8s node from service, to do this you can **drain** the node: containers running on the node will be gracefully terminated (and potentially rescheduled on another node).
To drain a node, use:
`kubectl drain <node name>`
When draining you may need to ignore `DaemonSets` (pods that are tied to each node). If you have any `DaemonSet` pods running on the node, you will likely need to use the `--ignore-daemonsets` flag:
`kubectl drain <node name> --ignore-daemonsets`
Once you are done with the maintenance of the node, `uncordon` the node:
`kubectl uncordon <node name>`
So containers and pods can start running again in that node.
## Upgrading with `kubeadm`
You can use `kubeadm` to keep your Kubernetes cluster. Here are the steps:
### Control plane upgrade steps
1. Upgrade `kubeadm` on the control plane.
2. Drain the control plane node.
3. Plan the upgrade with `kubeadm upgrade plan`.
4. Apply the upgrade with `kubeadm upgrade apply`.
5. Upgrade `kubelet` and `kubectl` on the control plane node.
6. Uncordon the control plane node.
Do you want the commands? Try this:
```
1.  sudo apt-get update && \ sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00
2.  kubectl drain k8s-control --ignore-daemonsets
3.  sudo kubeadm upgrade plan v1.27.2
4.  sudo kubeadm upgrade apply v1.27.2
5.  sudo apt-get update && \ sudo apt-get install -y --allow-change-held-packages 6.  kubelet=1.27.2-00 kubectl=1.27.2-00
7.  sudo systemctl daemon-reload
8.  sudo systemctl restart kubelet
9.  kubectl uncordon k8s-control
10. kubectl get nodes (until node shows ready)
```
### Worker node upgrade steps
1. Drain the node.
2. Upgrade `kubeadm`.
3. Upgrade the `kubelet` configuration `kubeadm upgrade node`.
4. Upgrade `kubelet` and `kubectl`.
5. Uncordon the node.
Do you want the commands? Try this:
```
On the control-plane node:
1.  kubectl drain k8s-worker1 --ignore-daemonsets --force
On the worker node:
2.  sudo apt-get update && \ sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00
3.  sudo kubeadm upgrade node
4.  sudo apt-get update && \ sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00
5.  sudo systemctl daemon-reload
6.  sudo systemctl restart kubelet
On the control-plane node:
7. kubectl uncordon k8s-worker2
```
## Backing Up and Restoring `etcd` Cluster Data
Why back up `etcd`? All your K8s objects, applications, and configurations are stored in `etcd`.

The right tool for the job: `etcdctl`

To back up the data use:
`ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save <file name>`
To restore the data use:
`ETCDCTL_API=3 etcdctl snapshot restore <file name>`

Do you want the commands? Here they are:
```
For creating the snapshot:
1. ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
  --endpoints=https://10.0.1.101:2379 \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key

For restoring the snapshot:
1. sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
  --initial-cluster etcd-restore=https://10.0.1.101:2380 \
  --initial-advertise-peer-urls https://10.0.1.101:2380 \
  --name etcd-restore \
  --data-dir /var/lib/etcd
2. sudo chown -R etcd:etcd /var/lib/etcd
```