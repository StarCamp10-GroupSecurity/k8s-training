# Learning Summary Kubernetes

## Table of Contents
- [Learning Summary Kubernetes](#learning-summary-kubernetes)
  - [Table of Contents](#table-of-contents)
  - [Kubernetes (K8s)](#kubernetes-k8s)
    - [Why Kubernetes?](#why-kubernetes)
    - [Kubernetes Architecture](#kubernetes-architecture)
      - [Master node(s)](#master-nodes)
      - [Worker Node(s)](#worker-nodes)
    - [Kubernetes Basic Component](#kubernetes-basic-component)
      - [Pods](#pods)
      - [ReplicaSet](#replicaset)
      - [Deployment](#deployment)
      - [Service](#service)
      - [Config Map](#config-map)
      - [Secret](#secret)
  - [Elastic Kubernetes Service (EKS)](#elastic-kubernetes-service-eks)
    - [EKS Architecture](#eks-architecture)
    - [Client Connection](#client-connection)
      - [Public Endpoint Access](#public-endpoint-access)
      - [Private Endpoint Access](#private-endpoint-access)
    - [Problems](#problems)
    - [EKS Price](#eks-price)
  - [Harness](#harness)
    - [Setup K8s Delegate (This is a part of CI Pipeline)](#setup-k8s-delegate-this-is-a-part-of-ci-pipeline)
    - [Setup Docker Delegate (Optional)](#setup-docker-delegate-optional)
    - [Create a CD Module in Harness](#create-a-cd-module-in-harness)
      - [Create a service](#create-a-service)
      - [Create environment](#create-environment)
      - [Configure the CD Pipline with the Service and Environment above](#configure-the-cd-pipline-with-the-service-and-environment-above)
- [Preferences](#preferences)

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

With the case of storing sensitive information above, we can use *Secret* to encode the sensitive information.

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
        - secretRef:
            name: sensitive-secret
        # env:
        # - name: SENSITIVE_VARIABLE
        #   value: thisisasecret
```

`secrets.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sensitive-secret
type: Opaque
data:
  SENSITIVE_VARIABLE: dGhpc2lzYXNlY3JldA==
```
## Elastic Kubernetes Service (EKS)

Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service that enables you to run Kubernetes seamlessly in both AWS Cloud and on-premises data centers.

Instead of build a Kubernetes Cluster from scratch, you can use EKS and deploy your application. EKS will help you manage the Control Plane(s), you have many options to deploy the worker nodes.

+ **Self-Managed Nodes:** Customers can launch their own EC2 instances and configure them to run as Kubernetes nodes in an EKS cluster. Self-managed nodes give customers more control over the EC2 instances but require more management overhead.

+ **Managed Node Groups:** Customers can also create managed node groups using the EKS console, CLI, or API. Managed node groups automatically provision and manage EC2 instances that run as Kubernetes nodes in an EKS cluster. They provide a simplified and scalable way to manage nodes in the cluster.

### EKS Architecture

An EKS cluster consists of **two VPCs**: **one VPC managed by AWS that hosts the Kubernetes control plane** and **a second VPC managed by customers that hosts the Kubernetes worker nodes (EC2 instances)** where containers run, as well as other AWS infrastructure (like load balancers) used by the cluster. All worker nodes need the ability to connect to the managed API server endpoint. This connection allows the worker node to register itself with the Kubernetes control plane and to receive requests to run application pods.

When you create a new EKS Cluster, there is an option: Choose the subnets in your VPC where the control plane may place elastic network interfaces (ENIs) to facilitate communication with your cluster. To create a new subnet, go to the corresponding page in the VPC console .

Below is the picture showing the option choosing subnet for placing ENI:

![Choosing subnet to place ENI](https://cyberdevops.s3.us-east-1.amazonaws.com/ChooseSubnets.png)

When we provision the EKS Control Plane and Worker Node, below is the architecture:

![High Level EKS Architecture](https://cyberdevops.s3.us-east-1.amazonaws.com/EasyEKSArchitect.png)

The order of operations for a worker node to come online and start receiving commands from the control plane is:

1. EC2 instance starts. Kubelet and the Kubernetes node agent are started as part of the boot process on each instance.
2. Kubelet reaches out to the Kubernetes cluster endpoint to register the node. It connects to the public endpoint outside of the VPC or to the private endpoint within the VPC.
3. Kubelet receives API commands and sends regular status and heartbeats to the endpoint. When you query the API server (kubectl get nodes), you see the latest status that each node’s Kubelet has reported back to the API server.

**Note**: Traffic from the private worker node to the AWS Control Plane is the private backbone, which means that it does not internet to communicate

### Client Connection

When creating a new Kubernetes Cluster, we have some options for Cluster Endpoint Access:

+ Public
+ Private
+ Public and Private

#### Public Endpoint Access

![Public Endpoint](https://cyberdevops.s3.us-east-1.amazonaws.com/endpoint_public.png)

#### Private Endpoint Access

![Private Endpoint](https://cyberdevops.s3.us-east-1.amazonaws.com/endpoint_private.png)

### Problems
1. When you launch your worker node (Node Group) in the private subnet, make sure that it can access the internet to download all the setup packages, connect to public registry such as Dockerhub
2. Normally, the IP address of the Kubernetes Cluster is different from the host. But in EKS, by default, EKS uses Amazon VPC CNI, it will assign an IP Adress (from Primary Subnet) to the pods. We have the problem of running out IP Address. Please refer to this [link](https://viblo.asia/p/giai-phap-cho-aws-eks-thieu-dia-chi-ipv4-hon-khat-nuoc-BQyJK3AMJMe)
### EKS Price

| Kubernetes Version Support Tier	 | Pricing |  
|----------|----------|
| Standard Kubernetes version support    | 	$0.10 per cluster per hour   |
| Extended Kubernetes version support    | $0.60 per cluster per hour   |

You also pay for the EC2 Instance (as a worker node). Please refer to this [link](https://aws.amazon.com/ec2/pricing/on-demand/) to see the plan for each type of instance.

## Harness

Prepare an EC2 for infrastructure. Your K8s cluster will be deployed on this EC2.

### Setup K8s Delegate (This is a part of CI Pipeline)

1. Make sure you have a K8s Cluster in the machine
2. Install a Harness Delegate in the Kind cluster

```sh
kubectl apply -f delegate.yaml -n harness-delegate-ng
```
3. Create a K8s connector

![Create a K8s Connector](https://cyberdevops.s3.us-east-1.amazonaws.com/k8s-connector.png)

### Setup Docker Delegate (Optional)

Refer to `harness/pool.yaml`, prepare an EC2, modify the security group ID and your subnet ID. Then, execute this command line

```
docker run -d -v /runner:/runner -p 3000:3000 drone/drone-runner-aws:latest  delegate --pool /runner/pool.yaml
```

Then, execute the command line below to run the Docker Delegate


```sh
docker run -d --cpus=1 --memory=2g --network host \
  -e DELEGATE_NAME=docker-delegate \
  -e NEXT_GEN="true" \
  -e DELEGATE_TYPE="DOCKER" \
  -e ACCOUNT_ID=WNy8fgXsSk2u8CRoTuwYSg \
  -e DELEGATE_TOKEN=YWU3M2YxZGIxNGFkMjVkNTczODcyYzA2ZDU1YWZmY2I= \
  -e DELEGATE_TAGS="" \
  -e MANAGER_HOST_AND_PORT=https://app.harness.io us-docker.pkg.dev/gar-prod-setup/harness-public/harness/delegate:25.06.86100
```

From now on, you can set up the CI pipeline for the project.

### Create a CD Module in Harness

Navigating to the `Continuous Delivery & GitOps`, and create a CD Pipeline. Below is the picture of a CD Pipeline:

![Create a K8s CD Pipeline](https://cyberdevops.s3.us-east-1.amazonaws.com/CDPipeline.png)

#### Create a service

A Harness service is what you're deploying.

A service has a service definition that contains its configuration details. These include manifest configurations, artifact configurations, and service variables.

In this case, we use Helm to deploy application, so please use Service Type with the `Manifest Details` as below:

![Create a Service Manifest Details](https://cyberdevops.s3.us-east-1.amazonaws.com/ManifestDetails.png)

#### Create environment

Create the `Pre-Prod environment` as below:

![Create a Pre-Prod environment](https://cyberdevops.s3.us-east-1.amazonaws.com/PreProd.png)

#### Configure the CD Pipline with the Service and Environment above

Create a Stage with the Deployment of `Native Helm` type.

1. Select the Service that you created above
2. Select the `Pre-Prod environment` above
3. Below is the simple CD pipeline

![Create a K8s CD Pipeline](https://cyberdevops.s3.us-east-1.amazonaws.com/CDPipeline.png)

# Preferences
1. [De-mystifying cluster networking for Amazon EKS worker nodes](https://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/)
2. [Amazon EKS pricing](https://aws.amazon.com/eks/pricing/)