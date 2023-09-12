## What is `kubeadm`?
It's a tool that will simplify the process of setting up our K8s cluster.
## Installing Packages
1. Create a configuration file for `containerd`:
`file: /etc/modules-load.d/containerd.conf`
```
overlay
br_netfilter
```
2. Use these to load the modules:
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
3. Set the system configuration for K8s networking:
`file: /etc/sysctl.d/99-kubernetes-cri.conf`
```
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
```
4. Apply the settings:
`sudo sysctl --system`
5. Install `containderd`:
`sudo apt update && sudo apt install -y containerd.io`
6. Create a default configuration file for `containerd`:
`sudo mkdir -p /etc/containerd`
7. Generate the default `containerd` configuration, and save it to the newly created default file:
`sudo containerd config default | sudo tee /etc/containerd/config.toml`
8. Restart `containerd` to ensure the new configuration file is used:
`sudo systemctl restart containerd`
9. Verify that `containerd` is running:
`sudo systemctl status containerd`
10. Disable swap:
`sudo swapoff -a`
11. Install the dependency packages:
`sudo apt update && sudo apt install -y apt-transport-https curl`
12. Add the GPG key:
`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`
13. Add K8s to the repository list:
`file: /etc/apt/sources.list.d/kubernetes.list`
```
deb https://apt.kubernetes.io/ kubernetes-xenial main
```
14. Update the package listings:
`sudo apt update`
15. Install the K8s package:
`sudo apt install -y kubelet=1.27.0-00 kubeadm=1.27.0-00 kubectl=1.27.0-00`
16. Turn off automatic updates for K8s:
`sudo apt-mark hold kubelet kubeadm kubectl`
## Initializing the Cluster
1. Initialize the K8s cluster on the control plane node using `kubeadm`:
`sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.0`
2. Set `kubectl` access:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
3. Test the access to the cluster:
`kubectl get nodes`
## Install the Calico Network Add-On
1. On the control plane node, install the Calico networking:
`kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml`
3. Check the status of the control plane node:
`kubectl get nodes`
## Join the Worker Nodes to the Cluster
1. In the control plane node, create the token and copy the `kubeadm join` command:
`kubeam token create --print-join-command`
2. Copy the full output from the previous command used in the control plane node. This command starts with `kubeadm join`.
3. In worker nodes, paste the full `kubeadm join` command to join the cluster. Use `sudo` to run it as root:
`sudo kubeam join...`
4. In the control plane node, view the cluster status:
`kubectl get nodes`