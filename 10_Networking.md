The K8s network model is a set of standards that define how networking between Pods behaves.

There are a variety of different implementations of this model, including the **Calico network plugin**, which we have been using throughout this course.

The K8s network model defines how Pods communicate with each other, regardless of which Node they are running on.

Each Pod has its own **unique IP address** within the cluster.

Any Pod can reach any other Pod using that Pod's IP address. This creates a **virtual network** that allows Pods to easily communicate with each other, regardless of which node they are on.

This is what you need to understand:

Essentially, the networking model makes the concept of K8s nodes completely transparent, Pods don't really need to be aware of what nodes other pods are running on in order to communicate with them across the network, they just need the pod's IP address, if two pods are in the same node, or if they are in completely different nodes, they can communicate with each other completely transparently using this virtual network.

![[Pasted image 20230909100141.png]]
## CNI Plugins

CNI plugins are a type of K8s network plugin. These plugins provide network connectivity between Pods according to the standard set by the K8s network model.

They implement the network model defined by K8s.

There are many plugins! How can I select one? Just check the K8s documentation for a list of available plugins. Each plugin has its own unique installation process.

**Important!**
K8s nodes will remain in `NotReady` until a network plugin is installed. You will be unable to run Pods while this is the case.

Check out the Calico plugin! It's well suited for most cases.
## Understanding K8s DNS

The K8s virtual network uses a **DNS** to allow pods to locate other pods and services using domain names instead of IP addresses.

This DNS runs as a service within the cluster. You can usually find it in the `kube-system` namespace.

`kubeadm` clusters use **CoreDNS** as they DNS solution.

### Pod Domain Names

All Pods in our `kubeamd` cluster are automatically given a domain name of the following form:
```
pod-ip-address.namespace-name.pod.cluster.local
```
A Pod in the default namespace with the IP address 192.168.10.100 would have a domain name like this:
```
192-168-10-100.default.pod.cluster.local
```

Here's how it looks:

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-dnstest
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dnstest
spec:
  containers:
  - name: nginx
    image: nginx:1.19.2
    ports:
    - containerPort: 80
```
## Using `NetworkPolicy`

A K8s `NetworkPolicy` is an object that allows you to control the flow of the network communication to and from pods. This allows you to build a more secure cluster network by keeping Pods isolated from traffic they do not need.

How it works? Using labels:

`podSelector`: Determines to which Pods in the namespace the `NetworkPolicy` applies. The `podSelector` can select Pods sing Pod labels.:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      role: db
```

**Note:** By default, Pods are considered non-isolated and completely open to all communication. If any `NetworkPolicy` selects a Pod, the Pod is considered isolated and will only be open to traffic allowed by `NetworkPolicies`.

A `NetworkPolicy` can apply to Ingress (incoming network traffic coming into the Pod), Egress (outgoing network traffic leaving the Pod) or both.

We also have from, to and IP block selectors:

- `from selector`: Selects ingress (incoming) traffic that will be allowed.
- `to selector`: Selects egress (outgoing) traffic that will be allowed.

They look like this:

```
spec:
  ingress:
    - from:
      ...
  egress:
    - to:
      ...
```

You can combine these selectors with `podSelector` (select pods to allow traffic from/to):

```
spec:
  ingress:
    - from:
      - podSelector:
          matchLabels:
            app: db
```

Also with `namespaceSelector` (select namespaces to allow traffic from/to):

```
spec:
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            app: db
```

And with `ipBlock` (select an IP range to allow traffic from/to):

```
spec:
  ingress:
    - from:
      - ipBlock:
          cidr: 172.17.0.0/16
```
### Ports

`port`: Specifies one or more ports that will allow traffic.

```
spec:
  ingress:
    - from:
      ports:
        - protocol: TCP
          port: 80
```

Traffic is only allowed if it matches both and allowed port and one of the from/to rules.