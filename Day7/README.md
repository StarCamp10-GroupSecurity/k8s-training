# Setup Harness Pipeline

## EC2

Prepare an EC2 for infrastructure. Your K8s cluster will be deployed on this EC2.

## Setup K8s Delegate

1. Make sure you have a K8s Cluster in the machine
2. Install a Harness Delegate in the Kind cluster

```sh
kubectl apply -f delegate.yaml -n harness-delegate-ng
```

## Setup Docker Delegate (Optional)

### Setup the Yaml file

Refer to `harness/pool.yaml`, prepare an EC2, modify the security group ID and your subnet ID. Then, execute this command line

```
docker run -d -v /runner:/runner -p 3000:3000 drone/drone-runner-aws:latest  delegate --pool /runner/pool.yaml
```

### Run the Docker Delegate

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