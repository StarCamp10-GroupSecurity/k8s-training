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

## Controller Manager

Controller will listen the request from the API Server and perform the action such as: create, read, update, and delte (CRUD)

There are some controllers that we need to know:
- Deployment controller
- Replicatioon Controller
- StatefulSet Controller
- etc.?

Examples of controllers include the Deployment controller, which creates new Pods based on a Deployment object’s spec, and the CronJob controller, which enables periodic creation of new Jobs.



## Scheduler

This component will decide which worker node will run the pods. It sounds very simple but it actually performs some complex algorithm to choose the most suitable worker node to run a pod.

Scheduler will listen the request from API Server: After **API Server** save a new pod in **etcd** but there is **no node attribute**, Scheduler will choose the worker node and **update the node attribute in etcd via API Server**.

# Elaboration on Worker node(s)

## Kubelet

In worker node, this is the component interacting with the master node(s). 

When we first join the worker node within the master node, kubelet will send a POST Request to the API Server to create a node resource.

If it receives a request from the API Server about creating a pod, it will create a container with the according container runtime.

## Kube proxy

As we know, the `service` has an endpoint, it will forward a traffic to pods. `Kube proxy` will handle the netwrok part so that a request can be forwarded to a pod


# Sample workflow creating a Nginx Deployment

We have a Nginx Deployment with Service as the file below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3 # number of the pod
  selector: 
    matchLabels:
      app: nginx # label value
  template: # pod template
    metadata:
      labels:
        app: nginx # label value
    spec:
      containers:
      - image: nginx:1.14.2 # image used to run container
        name: nginx # name of the container
        ports:
          - containerPort: 80 # pod of the container
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

Then we will the command `kubectl get events --watch --sort-by='.lastTimestamp'` to see all the events and finalize the workflow

![Workflow after running kubectl](https://cyberdevops.s3.us-east-1.amazonaws.com/workflow.png)


So, here is the workflow:

1. client perform `kubectl apply -f deployment.yaml`
2. `kubectl` will send a POST request to the API Server
3. `API Server` validate the configuration, if it is valid, it will save all resources in `etcd`
4. `API Server` send a request to the Deployment Controller
5. Deployment Controller receives the config files, it will find the field `template ReplicaSet` and send request to `ReplicaSet controller`
6. `ReplicaSet controller` receives the message and make the request to the `API Server` to create pods
7. `API Server` save the Pod resource to `etcd` and send a request to `Scheduler`
8. `Scheduler` picks the worker node to create pods and let the `API Server` know which worker node
9. `API Server` will send the request to `kubelet` in a Worker Node to create container.
10. `API Server` update the the information of pods in `etcd`



# Preferences
1. [Kubernetes Series - Bài 11 - Kubernetes internals architecture](https://viblo.asia/p/kubernetes-series-bai-11-kubernetes-internals-architecture-L4x5xPjb5BM)