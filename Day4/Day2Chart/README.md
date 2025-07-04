# Instruction

## Requirements
```
Install Nginx Ingress Controller via Helm: helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx.
Create an Ingress rule to route /app to the Day 2 S ervice. Example: ingress.yaml with kind: Ingress, spec.rules.path: /app.
Test with kubectl port-forward svc/ingress-nginx-controller 8080:80 and curl localhost:8080/app.
Package the chart, release new version, push to the registry
```

## Init the Helm Chart

```sh
kind create cluster --name day2cluster
```

## Fix the Chart

Copy from local 

## Install the chart

Check the template first:

```sh
helm template tamvo-test . -f values.yaml
```

```sh
helm install tamvo-test . -f values.yaml --namespace my-app
```

## Install the Ingress Nginx Controller

*Note*: They can be in different namespace

```sh
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx
```
Port forwarding nginx

## Push to Dockerhub

```sh
helm push day2-0.2.0.tgz oci://registry-1.docker.io/heyyytamvo
```