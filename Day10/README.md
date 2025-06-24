# Learning Summary Kubernetes

## Kubernetes (K8s)

### Why Kubernetes?

If you use container to deploy your application, there are some problem you have to face:

+ You application receives a large number of traffic and you have to scale a lot of containers to handle the resource
+ In case, your VM running out of memory, how do you deal with it? How to deploy more containers

And kubernetes is the answer for this issue.

### Kubernetes Architecture

A kubernetes cluster contains:
- 1 or multiple master nodes
- 1 or multiple worker nodes

Using `kubectl get pods -n kube-system`, we can see all the component of the kubernetes cluster as the picture below:

![Importan Component of a K8s Cluster](https://cyberdevops.s3.us-east-1.amazonaws.com/kube-system.png)

#### Master node(s)

A master node contains 4 components:
- etcd
- API server
- Controller Manager
- Scheduler

#### Worker Node(s)

A master node contains 4 components:
- kubelet
- kube-proxy
- container runtime

Please refer to Day 2 to see more details and how do they communicate to each others.

### Kubernetes Basic Component

#### Pods

The basic component of the kubernetes cluster is a pod. A pod can contains 1 or more containers. But usually, they deploy 1 container per pod. 

Remember, pod is **ephemeral**, which means that it can die and re-deploy. Every pod has an IP. Because it is ephemeral, so the IP address changes through time. That's why we can't use IP to communicate between pods.

Please refer to this picture for the sample demonstration:

![K8s Pods](https://cyberdevops.s3.us-east-1.amazonaws.com/pods.png)

Below is a simple `yaml` file to deploy a nginx pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - image: nginx:1.14.2  # Image used to run the container
    name: httpd          # Name of the container
    ports:
      - containerPort: 80 # Port exposed by the container
```
#### ReplicaSet

Now, if we would like to create multiple pods, we can use Replica Set to deploy as the `yaml` file below

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: httpd-rc
spec:
  replicas: 3 # number of the pod
  template: # pod template
    metadata:
      labels:
        app: httpd # label value
    spec:
      containers:
      - image: nginx:1.14.2 # image used to run container
        name: httpd # name of the container
        ports:
          - containerPort: 80 # pod of the container
```

With the config file above, you can create 3 pods and it will **make sure **that there are always 3 pods running in your kubernetes cluster.

The keyword is **make sure**. If one pod fails, the Replica Controller will restart it and make sure you have 3 healthy pods running.

Below is a the picture showing that a ReplicaSet will make sure there is 3 pods.

![K8s ReplicaSet](https://cyberdevops.s3.us-east-1.amazonaws.com/ReplicaSet.png)
#### Deployment

A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the **Deployment Controller** changes the actual state to the desired state at a controlled rate.

When you create a new deployment, it will create a new ReplicaSet. There are some updating methods:

+ Rolling Update
+ Recreate

Below is the deployment with a new image version:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
spec:
  replicas: 3 # number of the pod
  selector: 
    matchLabels:
      app: httpd # label value
  template: # pod template
    metadata:
      labels:
        app: httpd # label value
    spec:
      containers:
      - image: httpd:1.14.2 # new image used to run container: httpd instead of nginx
        name: httpd # name of the container
        ports:
          - containerPort: 80 # pod of the container
```

Below is the picture of a deployment (updating from nginx to httpd):

![K8s Deployment](https://cyberdevops.s3.us-east-1.amazonaws.com/Deployment.png)
#### Service

As we know, pods are *ephemeral*, which means that its IP Address changes if a new pod is deployed. Therefore, we will use Service to communicate between Pods and Pods or Client and Pods.

For example, if you are in the green pod and execute this command line:

```sh
curl http://serviceA
```

It will receive the response from 1 of 3 red pods. Traffic will be distributed to 1 of three red pods via Service A. The componet helping it is the `kube-proxy`.

![K8s Service](https://cyberdevops.s3.us-east-1.amazonaws.com/Service.png)

There are three kind of service:

+ Cluster IP
+ Node Port
+ Load Balancer (Just support in the cloud environment: EKS, AKS, GKE, etc.)

Node Port and Cluster IP is the same, but for the service Node Port, you specify the port of the Node manually.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  labels:
    app: httpd  # Service will be forward to all pods with label of `app: httpd`
spec:
  type: ClusterIP
  selector:
    app: httpd
  ports:
  - port: 80    # port of the service
    targetPort: 80   # port of container
```
#### Config Map

ConfigMaps are used to store non-sensitive configuration data in key-value pairs. This can include application settings, environment variables, command-line arguments, or configuration files.

Below is the a deployment without using config map. When everything is deployed successfully, you will exec to the pods and see the expected environment variable.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
spec:
  replicas: 3 # number of the pod
  selector: 
    matchLabels:
      app: httpd # label value
  template: # pod template
    metadata:
      labels:
        app: httpd # label value
    spec:
      containers:
      - image: httpd:1.14.2 # new image used to run container: httpd instead of nginx
        name: httpd # name of the container
        ports:
          - containerPort: 80 # pod of the container
        env:
        - name: SENSITIVE_VARIABLE
          value: thisisasecret
```

If you use config map, there will be 2 files.

`deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
spec:
  replicas: 3 # number of the pod
  selector: 
    matchLabels:
      app: httpd # label value
  template: # pod template
    metadata:
      labels:
        app: httpd # label value
    spec:
      containers:
      - image: httpd:1.14.2 # new image used to run container: httpd instead of nginx
        name: httpd # name of the container
        ports:
          - containerPort: 80 # pod of the container
        envFrom:
        - configMapRef:
            name: sensitive-config
        # env:
        # - name: SENSITIVE_VARIABLE
        #   value: thisisasecret
```

`config_map.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sensitive-config
data:
  SENSITIVE_VARIABLE: thisisasecret
```
#### Secret

#### Persistent Volume and Persistent Volume Claim

## Elastic Kubernetes Service (EKS)

## Helm 

## Harness