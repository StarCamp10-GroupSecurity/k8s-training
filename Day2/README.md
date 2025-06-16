# Kubernetes Architecture

A kubernetes cluster contains:
- 1 or multiple master nodes
- 1 or multiple worker nodes

Using `kubectl get pods -n kube-system`, we can see all the component of the kubernetes cluster as the picture below:

![Importan Component of a K8s Cluster](https://cyberdevops.s3.us-east-1.amazonaws.com/kube-system.png)

# Master node(s)

A master node contains 4 components:
- etcd
- API server
- Controller Manager
- Scheduler
  
# Worker Node(s)

A master node contains 4 components:
- kubelet
- kube-proxy
- container runtime

# How do they interact to each other?

All component in a kubernetes cluster **is not running as a process**. They run as pods and they are deployed to master node or worker node.

*Note:* **kubelet** is the only component which run as an application in the worker node (run in a process)

Below is the picture where I ssh to the worker node and show all the runnng process:

![A kubelet process in a Worker Node](https://cyberdevops.s3.us-east-1.amazonaws.com/kubelet.png)

All components interact through API Server and **API Server is the only component interacting with etcd**

In worker node, container runtime is managed by the kubelet. Worker node and master node communicate via kubelet

![A kubelet process in a Worker Node](https://cyberdevops.s3.us-east-1.amazonaws.com/master-worker.png)

# Elaboration on Master node(s)

## Etcd

Etcd is a **database of a kubernetes cluster**. It is simple, how do kubernetes know how many pods, services, resources, etc. in the cluster. All information is store in the etcd.

## API Server

This component is the heart of the system. Other components or client will interact with API Server to perform action on the cluster. After that, the API Server will update the state of the cluster on etcd.

When using `kubectl`, we actually send a HTTP Request to the API Server. Also, API Server will perform the authentication checking.

![Working with kubectl](https://cyberdevops.s3.us-east-1.amazonaws.com/kubectl-work.png)