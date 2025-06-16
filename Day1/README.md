# Requirements

```
Kind cluster running, kubectl get nodes shows at least 1 node.
Namespace my-app created.
Nginx Pod running, kubectl get pods -n my-app shows nginx-pod in Running state.
kubectl describe pod shows pod details.
Can describe about K8s architecture, know how the component talk to each others.
Know how to access cluster, switch namespace.
```

# Instruction
Make sure that your Ubuntu runs this bash script file here to [install](https://github.com/StarCamp10-GroupSecurity/k8s-training/blob/master/install.sh) all dependency

## Create the k8s cluster with kind
```sh
kind create cluster --name k8s-playground --config kind-config.yaml
```

### Create the pod and namespace
You can see the `k8s` folder, navigate to it and perform the following command:

```sh
kubectl apply -f .
```