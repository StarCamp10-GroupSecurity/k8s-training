# Requirements

```
Learn Persistent Volumes (PVC) and StorageClasses.
Understand Ingress for HTTP routing.
Set up an Ingress Controller.
Create a PVC for a Nginx pod
Example: pvc.yaml with kind: PersistentVolumeClaim, spec.storageClassName: standard.
Deploy Nginx with PVC and test data persistence ( custom index.html in host path or another zombie file saving in PVC is OK ? )
```

This file is for learning pvc and pv. There are two worker node, but this config file makes sure that a pod is deployed within a specific worker node to bind the volume.

