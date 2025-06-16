# Requirements
```
HPA scales pods up/down, kubectl get hpa shows current/desired replicas.
Liveness probe restarts pod on failure, kubectl describe pod shows probe events.
Troubleshooting identifies bad image tag, fixed pod runs.
Guestbook app deployed from Git repo, accessible via Service.
kubectl events shows cluster activity.
```

# HPA
I use the `hpa.yaml` file but cant test the scaling up/down

# Probes

## Liveness
Apply all the files in `probesDay2/` and perform `kubectl get pods` to see its status

## Modifying a dummy image and troubleshoot

There are multiple techniques to troubleshoot:

### Describe Pod
```
kubectl describe pod 
```

### Using event
```
kubectl get events --watch --sort-by='.lastTimestamp
```

## Updating the probe to see different behaviour


