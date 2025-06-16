# Kubernetes Architecture

A kubernetes cluster contains:
- 1 or multiple master nodes
- 1 or multiple worker nodes

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

Tất cả những component trong một kubernetes cluster sẽ không phải chạy như một ứng dụng cài trực tiếp trên server.

All component in a kubernetes cluster is not running in a process. They run as pods and they are deployed to master node or worker node.

*Note:* **kubelet** is the only component which run as an application in the worker node (run in a process)

Below is the picture where I ssh to the worker node and show all the runnng process:

![A kubelet process in a Worker Node](https://cyberdevops.s3.us-east-1.amazonaws.com/kubelet.png)