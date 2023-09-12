## Managing Application Configuration

You may want to pass dynamic values to your applications at runtime to control how they behave. This is known as **application configuration**.
### `ConfigMaps`

You can store configuration data using `ConfigMaps`, those store data in a key-value map. `ConfigMap` data can be passed to your container applications.

Check this example:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: value1
  key2: value2
  key3:
    subkey:
      morekeys: data
      evenmore: some more data
  key4: |
    You can also do
    multi-line
    data.
```
### `Secrets`

`Secrets` are very similar to `ConfigMaps` but are designed to store sensitive data, such as passwords or API keys, more securely:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: user     <- values must be encoded using base64
  password: mypass
```
### `Passing the data to the container`

You can pass `ConfigMap` and `Secret` data to your containers as **environment variables**. These will be visible to your container process at runtime. Here's how you would use them:

```
spec:
  containers:
  - ...
    env:
    - name: ENVVAR
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: mykey
```

At your container, you'll be able to access this data using `$ENVVAR` runtime environment variable.

You can also pass `ConfigMap` or `Secret` data to your container using **mounted volumes**. This will cause the configuration data to appear in files available to the container file system:

```
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - name: configmap-volume
      mountPath: /etc/config/configmap
    - name: secret-volume
      mountPath: /etc/config/secret
  volumes:
  - name: configmap-volume
    configMap:
      name: my-configmap
  - name: secret-volume
    secret:
      secretName: my-secret
```

Each top-level key in the configuration data will appear as a file containing all keys below that top-level key:
```
...
volumes:
- name: secret-vol
  secret: secretName: my-secret
```
## Managing Container Resources

**Resource requests** allow you to define an amount of resources (such as CPU or memory) you expect a container to use. The K8s scheduler will use resources requests to avoid scheduling pods on nodes that do not have enough available resources.

Tip: containers are allowed to use more or less resources, resource requests only affect scheduling!

Here's how they look:
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
```

**Resource limits** provide a way for you to limit the amount of resources your containers can use. The container runtime is responsible for enforcing these limits, and different container runtimes do this differently.

Tip: some runtimes will enforce these limits by terminating the container processes that attempt to use more than the allowed amount of resources.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "264Mi"
```
## Monitoring Container Health

K8s provides features like automatically restarting unhealthy containers, it needs to know how to determine the status of your applications.
### Liveness probes

**Liveness probes** allow you to automatically determine whether or not a container application is in a healthy state. By default, K8s considers a container to be "down" if the container process stops.
With liveness probes, you can customize this detection mechanism and make it more sophisticated.

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    livenessProbe:
      exec:
        command: ["echo", "Hwllo World"]
      initialDelaySeconds: 5
      periodSeconds: 5
```
### Startup Probes

They are very similar to liveness probes. However, while liveness probes run constantly on a schedule, startup probes run at container startup and stop running once they succeeded.
Essentially, startup probes are used to detect when the application has successfully started up.

```
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
```
### Readiness Probes

They are used to determine when a container is ready to accept requests. When you have a service backed by multiple container endpoints, user traffic will not be sent to a particular pod until its containers have all passed the readiness checks defined by their readiness probes.
Essentially, they detect when the container is fully started up and prevent user traffic from being sent to pods that are not ready yet.

```
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```
## Building Self-Healing Pods with Restart Policies

K8s can automatically restart containers when they fail. **Restart policies** allow you to customize this behavior by defining when you want a pod's containers to be automatically restarted.
There are three possible values for a pod's restart policy: `Always`, `OnFailure` and `Never`:

- `Always`
It is the default restart policy. Containers will always be restarted if they stop, even if they completed successfully. Use this for applications that should always be running.

- `OnFailure`
Containers will be restarted if a process exits with an error code or the container is determined to be unhealthy by a liveness probe. Use this for applications that need to run successfully and then stop.

- `Never`
Containers will never be restarted, even if the container exits or a liveness probe fails. Use for applications that should run once and never be automatically restarted.

Usage:
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 10']
  restartPolicy: <Always | OnFailure | Never>
```

### Creating Multi-Container Pods

A pod with more than one container is a multi-container pod. Containers sharing the same Pod can interact with one another using shared resources, some of these are:

- Network
Containers share the same networking namespace and can communicate with one another or any port, even if that port is not exposed to the cluster.

- Storage
Containers can use volumes to share dat in a Pod.

This is how they look like:
```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo logs data > /output/output.log; sleep 5; done']
    volumeMounts:
    - name: sharedvol
      mountPath: /output
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /input/output.log']
    volumeMounts:
    - name: sharedvol
      mountPath: /input
  volumes:
  - name: sharedvol
    emptyDir: {}
```
## Init Containers

These are containers that run once during the startup process of a pod. A pod can have any number of init containers, and they will each run once (in order) to completion.

![[Pasted image 20230903200433.png]]

Use them to perform a variety of startup tasks, they can contain and use software and setup scripts that are not needed by your main containers, so you can keep your main containers lighter and more secure by offloading startup tasks. Here are some use cases:

- Cause a pod to wait for another K8s resource to be created before finishing startup.
-  Perform sensitive startup steps securely outside of app containers.
- Populate data into a shared volume at startup.
- Communicate with another service at startup.

Here is some example:
```
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
  initContainers:
  - name: delay
    image: busybox
    command: ['sleep', '30']
```
