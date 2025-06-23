# Harness CD with Helm Charts	

## Create a CD Module in Harness
Navigating to the `Continuous Delivery & GitOps`, and create a CD Pipeline. Below is the picture of a CD Pipeline:

![Create a K8s CD Pipeline](https://cyberdevops.s3.us-east-1.amazonaws.com/CDPipeline.png)

## Create a service

A Harness service is what you're deploying.

A service has a service definition that contains its configuration details. These include manifest configurations, artifact configurations, and service variables.

In this case, we use Helm to deploy application, so please use Service Type with the `Manifest Details` as below:

![Create a Service Manifest Details](https://cyberdevops.s3.us-east-1.amazonaws.com/ManifestDetails.png)

## Create environment

Create the `Pre-Prod environment` as below:

![Create a Pre-Prod environment](https://cyberdevops.s3.us-east-1.amazonaws.com/PreProd.png)

## Configure the CD Pipline with the Service and Environment above

Create a Stage with the Deployment of `Native Helm` type.

1. Select the Service that you created above
2. Select the `Pre-Prod environment` above
3. Below is the simple CD pipeline

![Create a K8s CD Pipeline](https://cyberdevops.s3.us-east-1.amazonaws.com/CDPipeline.png)