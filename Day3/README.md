# Requirements

```
Helm installed, WordPress deployed via Helm chart, accessible via kubectl port-forward.
ConfigMap and Secret applied, kubectl get configmaps,secrets -n my-app lists them.
kubectl exec into Day 2 pod confirms APP_ENV and password are set.
helm list -n my-app shows WordPress release.
```

# Instruction
## Install Helm
Make sure your ubuntu run the bash script [here](https://github.com/StarCamp10-GroupSecurity/k8s-training/blob/master/install.sh).

## Create Deployment, Service, Config Map, and Secret in the folder Day2Update

```sh
cd Day2Update
kubectl create namespace my-app
kubectl apply -f . -n my-app
```

## Verify all environment variables inside a pod

```sh
kubectl exec -it <pod-name> -- bash
export
```

## Using helm to install Wordpress

```sh
kubectl create namespace my-app
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wordpress bitnami/wordpress -n my-app
```

## Verify the wordpress

```sh
kubectl get pods -n my-app
```

## Migrate Day2 to Helm Chart
Using the helm folder [here](https://github.com/StarCamp10-GroupSecurity/k8s-training/tree/master/Day3/helm).

### Install the helm chart
```sh
helm install tamvo ./ --namespace new --create-namespace
```

### Login to your dockerhub
```sh
docker login -u <your-username>
```

### Push the chart to dockerhub
```sh
helm package .
helm push day2-0.1.0.tgz oci://registry-1.docker.io/<your-username>
```

### Pull the chart to test the result 
```sh
helm install testpull oci://registry-1.docker.io/<your-username>/day2 --version 0.1.0
```

You can see my Repository in Dockerhub [here](https://hub.docker.com/repository/docker/heyyytamvo/day2/general).