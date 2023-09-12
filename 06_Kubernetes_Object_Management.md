## Working with `kubectl`
`kubectl` is a CLI that allows you to interact with K8s. It uses the K8s API to communicate with the cluster and carry out your commands.

### `kubectl get`
List objects in the cluster:
```
kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
```
- `-o`: set output format.
- `--sort-by`: sort output using a `JSONPath` expression.
- `--selector`: filter results by label.
### `kubectl describe`
Get detailed information about K8s object:
```
kubectl describe <object type> <object name>
```
### `kubectl create`
Create K8s objects:
```
kubectl create -f <file name>
```
Supply a YAML file with `-f` to create an object from a YAML descriptor stored in a file.
If you try to create an object that already exist, an error will occur.
### `kubectl apply`
This is similar to `kubectl create`. However, if you use `kubectl apply` on an object that already exist, it will modify the existing object, if possible.
```
kubectl apply -f <file name>
```
### `kubectl delete`
Delete objects from the cluster:
```
kubectl delete <object type> <object name>
```
### `kubectl exec`
Run commands inside containers.
```
kubectl exec <pod name> -c <container name> -- <command>
```
## Tips for `kubectl`
You can create of modify K8s objects using declarative or imperative ways:
- Declarative: define objects using data structures such as YAML or JSON.
- Imperative: define objects using `kubectl` commands and flags.
### How to get quick sample YAML?
Use the `--dry-run` flag to run an imperative command without creating an object. Combine it with `-o yaml` to quickly obtain a sample YAML file you can manipulate!
`kubectl create deployment my-deployment --image=nginx --dry-run -o yaml`
### Record a command
Use the `--record` flag to record the command that was used to make a change.
`kubectl scale deployment my-deployment replicas=5 --record`
## Managing K8s Role-Based Access Control
RBAC in K8s allows you to control what users are allowed to do and access within your cluster. For example, you can use RBAC to allow developers to read metadata and logs from K8s pods but not make changes to them.
### `Roles` and `ClusterRoles`
`Roles` and `ClusterRoles` are K8s objects that define a set of permissions, these permissions determine what users can do in the cluster.
- A `Role` defines permissions within a particular namespace!!!
- A `ClusterRole` defines cluster-wide permissions not specific to a single namespace!!!
### How do you connect these roles with actual users?
`RoleBinding` and `ClusterRoleBinding` are objects that connect users to roles.
![[Pasted image 20230901120812.png]]
## Creating Service Accounts
In K8S, a service account is an account used by container processes within pods to authenticate with the K8s API.
**IF YOUR PODS NEED TO COMMUNICATE WITH THE K8S API, YOU CAN USE SERVICE ACCOUNTS TO CONTOL THEIR ACCESS.**

Bind service accounts with `RoleBindings` or `ClusterRoleBindings` to provide access to K8s API functionality.

This is how a service account looks like:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
```
This is how a binding for service account would look like:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-pod-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```
## Inspecting Pod Resource Usage
In order to view metrics about the resources pods and containers are using, we need an add-on to collect and provide that data. One such add-on is **Kubernetes Metrics Server**.

To install it you can do:
`kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml`

One you have that add-on installed, you can use commands like `kubectl top`, to view data about resource usage in your pods and nodes:
`kubectl top pod --sort-by <JSONPATH> --selector <selector>`