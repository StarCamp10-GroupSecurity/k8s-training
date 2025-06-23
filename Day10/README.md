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

#### Deployment

#### Service

#### Config Map

#### Secret

#### Persistent Volume and Persistent Volume Claim

## Elastic Kubernetes Service (EKS)

## Helm 

## Harness