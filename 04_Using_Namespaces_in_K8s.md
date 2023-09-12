What is a namespace?
Virtual clusters backed by the same physical cluster. Pods and containers live in namespaces, it's the way K8s separates and organizes itself.

- List existing namespaces using:
`kubectl get namespaces`
- All cluster have a default namespace, this is used when no other namespace is specified.
- `kubeadm` also creates a `kube-system` namespace for system components.
- When using `kubectl` you can specify the namespace using `--namespace`: `kubectl get pods --namespace my-namespace`, the short version is `-n`
- Do you want objects from all namespaces? Use: `kubectl get pods --all-namespaces`
- Create namespaces using: `kubectl create namespace my-namespace`
