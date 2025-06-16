# Requirements
```
Deployment running with 3 httpd pods, kubectl get pods shows 3 pods.
ClusterIP and NodePort Services created, kubectl get services lists them.
curl localhost:8080 shows Apache welcome page via port-forward.
Rolling update completes without downtime, kubectl rollout status confirms.
```

# Instruction
## Create deployment and service in default namespace

```sh
kubectl apply -f Deployment.yaml Service.yaml 
```
## Show all running pods in default namespace

```sh
kubectl get pods
```

## Check the app with port-forwarding
```sh
kubectl port-forward svc/<your-service> 8080:80
```

Open another session and perform `curl`

```sh
curl localhost:8080
```

## Update the image and see the Rolling Update

Modify the image from `httpd` to `nginx`. Use the following command line to check the result

```sh
kubectl rollout status
```

