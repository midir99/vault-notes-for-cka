K8s services provide a way to expose an application running as a set of Pods.
They provide an abstract way for clients to access applications without needing to be aware of the application's Pods.
## Service Routing

Client make requests to a Service, which **routes** traffic to its Pods in a load-balanced fashion.
## Endpoints

**Endpoints** are the backend entities to which Services route traffic. For a Service that routes traffic to multiple Pods, each Pod will have an endpoint associated with the Service.

Tip: One way to determine which Pod(s) a Service is routing traffic to is to look at that service's Endpoints.
## Using K8s Services
### Service Types

Each service has a type. The **Service type** determines how and where the Service will expose your application. There are four service types:

- `ClusterIP`

These Services expose applications inside the cluster network. Use them when your clients will be other Pods within the cluster.

![[Pasted image 20230911221423.png]]

- `NodePort`

These services expose applications outside the cluster network. Use them when applications or users will be accessing your application from outside the cluster.

![[Pasted image 20230911221532.png]]

- `LoadBalancer`

These also expose applications outside the cluster network, but they use an external cloud load balancer to do so. This service type only works with cloud platforms that include load balancing functionality.

![[Pasted image 20230911221638.png]]

- `ExternalName` (outside the scope of CKA)
### Hands-On Demostration

1. First let's create a Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-svc-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: svc-example
  template:
    metadata:
      labels:
        app: svc-example
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```
2. Then build a Service that routes traffic to those Pods:
```
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  type: ClusterIP
  selector:
    app: svc-example
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
3. Get the endpoints of your Service:
`kubectl get endpoints svc-clusterip`

Tip: You can have any number of services routing traffic to the same Pods.