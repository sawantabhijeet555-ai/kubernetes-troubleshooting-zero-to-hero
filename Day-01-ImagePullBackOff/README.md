# Day 01: ImagePullBackOff

## What is ImagePullBackOff?

`ImagePullBackOff` occurs when Kubernetes cannot pull a container image from a registry. The container remains in the `Waiting` state while Kubernetes retries the image pull.

### Common Causes
- Invalid image name or tag.
- Pulling a private image without a valid `imagePullSecret`.

### How Does BackOff Work?

Kubernetes retries pulling the image with an increasing delay between attempts, up to a maximum of 300 seconds (5 minutes).

### Troubleshooting Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

Check the Pod Events to identify the root cause.


