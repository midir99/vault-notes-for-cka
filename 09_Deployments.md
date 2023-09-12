A **deployment** is a K8S object that defines a desired state for a `ReplicaSet` (a set of replica pods). The deployment controller seeks to maintain the desired state by creating, deleting, and replacing pods with new configurations.

Put it this way:

- You create a deployment with 3 replicas.
- Deployment controller will create those 3 pods in order to maintain that desired state.

A deployment desired state includes:

1. `replicas`: the number of replica pods the deployment controller will seek to maintain.
2. `selector`: a label used to identify the replica pods managed by the deployment.
3. `template`: a template pod definition used to create replica pods.

Some use cases for deployments are:

1. Easily scale an application up or down by changing the number of replicas.
2. Perform rolling updates to deploy a new software version.
3. Roll back to a previous software version.

This is how a deployment looks:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```

## Scaling Applications with Deployments

**Scaling** refers to dedicating more (or fewer) resources to an application in order to meet changing needs.

K8s deployments are very useful in **horizontal scaling**, which involves changing the number of containers running an application. If the `replicas` number is changed, replica pods will be created or deleted to satisfy the new number.

How to scale a deployment?

You could change the deployment simply by changing the number of `replicas` in the YAML descriptor with `kubectl apply` or `kubectl edit deployment <DEPLOYMENT NAME>`.

```
...
spec:
  replicas: 5
  ...
```

Or you could use `kubectl scale deployment.v1.apps/my-deployment --replicas 4`.

## Managing Rolling Updates with Deployments

**Rolling updates** allow you to make changes to a deployment's pods at a controlled rate, gradually replacing old pods with new pods. This allows you to update your pods without incurring downtime.

The steps would be like the following:
1. Use `kubectl` to edit your deployment to use the new version of the app.
2. Then use `kubectl rollout status deployment.v1.apps/<DEPLOYMENT NAME>`
3. In case of rollback, use `kubectl rollout undo deployment.v1.apps/<DEPLOYMENT NAME>`
### Rollback

If an update to a deployment causes a problem, you can **roll back** the deployment to a previous working state.