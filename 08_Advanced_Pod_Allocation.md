## What is Scheduling?
It is the process of assigning Pods to Nodes so `kubelets` can run them.

What is the **scheduler**?
It is a control plane component that handles scheduling.
### The Scheduling Process
The K8s scheduler selects a suitable Node for each Pod, it takes into account things like:
- Resource requests vs. available node resources.
- Various configurations that affect scheduling using node labels.
So we can customize the scheduling process to select or refuse to select certain nodes based on a variety of factors, usually node labels.

One special configuration is the `nodeSelector`, you can configure it for your Pods to limit which Node(s) the Pod can be scheduled on.

Node selectors user node labels to filter suitable nodes:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    myLabel: myvalue
```

The above tells the scheduler to schedule this pod on nodes that have that label with that value.

Another technique you should be aware of is `nodeName`, it allows you to bypass scheduling and assign a Pod to a specific Node by name using `nodeName`.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: k8s-worker1
```
## Using `DaemonSets`
A `DaemonSet` automatically runs a copy of a Pod on each node. `DaemonSets` will run a copy of the Pod on new nodes as they are added to the cluster.

`DaemonSets` respect normal scheduling rules around node labels, taints, and tolerations. If a pod would not normally be scheduled on a node, a `DaemonSet` will not create a copy of the Pod on that Node.

Here is an example:
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-daemonset
  template:
    metadata:
      labels:
        app: my-daemonset
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
```

Some brief explanations:

- `selector`: It allows the `DaemonSet` to identify which pods are being managed by the `DaemonSet`, so pods have to match a selector in order to be identified as pods that this `DeamonSet` is managing.
## Using Static Pods
A static pod is a pod managed directly by the `kubelet` on a node, not by the K8s API server. They can run even if there is no K8s API server present.

`kubelet` automatically creates static pods from YAML manifest files located in the manifest path on the node. You can specify where is the location of these files with `kubelet` configuration, the default location for these is: `/etc/kubernetes/manifests/my-static-pod.yml`, the contents of these files are a very straightforward pod:
```
apiVerison: v1
kind: Pod
metadata:
  name: my-static-pod
spec:
  containers:
  - name: nginx
    image: nginx: 1.19.1
```

`kubelet` will actually check that directory and it will automatically see the file and it will create that static pod, but if you don't want to wait just run: `sudo systemctl restart kubelet`

Mirror pods?
`kubelet` will create a **mirror Pod** for each static Pod. Mirror Pods allow you to see the status of the static Pod via the K8s API, but you cannot change or manage them via the API.
## Tips

- Label nodes or resources using `kubectl label <resource type> <resource name> <label key>=<label value>`