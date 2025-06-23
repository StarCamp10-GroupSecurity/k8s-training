# Harness Pipeline Automation

## Setup a trigger in the project

### Create a connector to the repository
In Harness, create a connector to your github repository. In my case, here is the configuration:

+ URL: https://github.com/StarCamp10-GroupSecurity/k8s-training
+ Authentication: UsernameToken
+ Username: heyyytamvo
+ Persontal Access Token: Create a secret with your password
+ API Authentication: Create a secret with your API Token

### Create a trigger in your pipeline

1. Navigating to `Organization: default -> Project: Default Project -> Pipelines -> Triggers`, add a trigger to your pipeline.

2. Select `GitHub` option
3. Naming your Trigger and select the Connector that you created above.
4. Select Event `Push`
5. Configure the API Authentication Secret

## Testing

Push code to your repository and see the pipeline run.

![Create a K8s Connector](https://cyberdevops.s3.us-east-1.amazonaws.com/webhook.png)

## Setup variable to manipulating deployment

### Background

Since we use Helm to deploy the K8s Cluster, instead of using the default `values.yaml` file, we will use `customValues.yaml` to manipulate the deployment (e.g change the number of replica set, etc.)

### Custom file and variable

Go to `Project Settings > File Store`, create file `replica_values.yaml`, replace its content with the file `customValues.yaml` file.

Below is its content:

```yaml
replicaCount: <+pipeline.variables.replica_count>
image:
  registry: docker.io
  repository: bitnami/nginx
  tag: 1.28.0-debian-12-r3
```

As you can see, it uses the variable in the pipeline. Go to the pipeline, setting up the `replica_count` variable as the picture below:

![Pipeline Variable](https://cyberdevops.s3.us-east-1.amazonaws.com/variable.png)

### Modify the service to use the custom file

Remember the service for this deployment in Harness? Go to its Configurations and add a new manifest `replica_values.yaml` as picture below:

![Pipeline Variable](https://cyberdevops.s3.us-east-1.amazonaws.com/addManifest.png)

Now, the CD Pipeline can work with the variable to manipulate the deployment.