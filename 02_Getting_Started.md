- Kubernetes is a platform to manage container workloads
- What does **K8s** mean? It's a short hand for Kubernetes
## What K8s actually does?

Kubernetes builds an abstraction to servers running containers, it calls it cluster.
You don't worry about which container will be running the container, you just run it and let the cluster run the container. They key features are:
- Container orchestration: it manages the containers
- Application reliability: it's easy to build self-healing apps using it
- Automation: you can automate the management of your containers
## Parts of K8s
### Control Plane
A collection of multiple components responsible for managing the cluster itself globally. It controls the cluster. These can run on any machine, usually are run on dedicated controller machines. It consists of:
- `kube-api-server`: servers the K8s API, the interface to the control plane and the cluster.
- `etcd`: the backend data store, it provides high-availability storage for all data related to the state of the cluster.
- `kube-scheduler`: handles the process of selecting an available node in the cluster on which to run containers.
- `kube-controller-manager`: runs a collection of controller utilities in a single process, these carry out automation-related tasks within the Kubernetes cluster.
- `cloud-controller-manager`: it provides an interface between K8s and cloud platforms (AWS, Azure, GCP, etc...), only used when using cloud-based resources alongside Kubernetes.
![[Pasted image 20230826222421.png]]
### Nodes
Kubernetes nodes are the machines where the containers managed by the cluster run. A cluster can have N number of nodes. Various node components manage containers on the machine and communicate with the control plane, like:
- `kubelet`: K8s agent that runs on each node. It communicates with the control plane and ensures that containers are run on its node as instructed by the control plane. It also handles the process of reporting container status and other data about container back to the control plane.
- `container runtime`: it is not built into K8s, it's a separate piece of software that is responsible for actually running containers on the machine. For example: K8s supports `Docker` and `containerd`.
- `kube-proxy`: is a network proxy, it runs on each node and handles tasks related to providing networking between containers an services in the cluster.
### The big picture
![[Pasted image 20230826223342.png]]