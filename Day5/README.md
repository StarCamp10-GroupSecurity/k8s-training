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

### Startup Probe

Startup probes are used to determine whether an application has started successfully. They are useful for applications that may take a long time to start.

```yaml
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
        startupProbe:
          httpGet:
            path: /healthz               # Endpoint to check
            port: 80                     # Port to access
          initialDelaySeconds: 30        # Wait before starting checks
          periodSeconds: 10               # Check every 10 seconds
          failureThreshold: 5             # Restart after 5 failures
```

### Readiness Probe

Readiness probes determine whether a container is ready to accept traffic. If a readiness probe fails, the container will be removed from the service’s endpoints until it passes.

```yaml
spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /ready                # Endpoint to check
            port: 80                    # Port to access
          initialDelaySeconds: 5         # Wait before starting checks
          periodSeconds: 5                # Check every 5 seconds
          failureThreshold: 3             # Mark as not ready after 3 failures
```