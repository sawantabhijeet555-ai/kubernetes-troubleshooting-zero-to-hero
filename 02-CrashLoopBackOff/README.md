# CrashLoopBackOff in Kubernetes

## 1. What is CrashLoopBackOff?

`CrashLoopBackOff` is a Kubernetes status indicating that a container repeatedly starts, crashes, and gets restarted. Kubernetes waits for an increasing amount of time between restart attempts using an exponential backoff mechanism.

The kubelet attempts to restart the container according to the Pod's restart policy. If the container continues to fail, Kubernetes keeps retrying with increasing delays.

**Important:** `CrashLoopBackOff` is not the root cause of the problem. It is an indication that the container is repeatedly failing to run successfully.

## 2. Common Causes of CrashLoopBackOff

### 2.1 Misconfiguration

Incorrect environment variables, configuration files, database credentials, or volume mounts can prevent an application from starting.

**Example:**

An application requires a database connection string, but the environment variable is missing.

```yaml
env:
  - name: DB_HOST
    value: ""
```

**Solution:**

- Verify environment variables and ConfigMaps.
- Check Secrets and database credentials.
- Verify volume mounts and configuration file paths.

### 2.2 Incorrect Liveness Probe

A liveness probe checks whether an application is running properly. If it repeatedly fails, Kubernetes restarts the container.

**Example:**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

If the application does not expose `/health` on port `8080`, the probe may fail repeatedly.

**Solution:**

- Verify the health-check endpoint and port.
- Adjust `initialDelaySeconds`, `periodSeconds`, and `failureThreshold` as needed.
- Use a startup probe for applications that require more time to initialize.

### 2.3 Insufficient Memory Limit

If a container exceeds its memory limit, the Linux kernel may terminate it with an out-of-memory (OOM) kill. Kubernetes may then restart it.

**Example:**

```yaml
resources:
  requests:
    memory: "64Mi"
  limits:
    memory: "64Mi"
```

**Solution:**

- Check whether the container was terminated due to an OOM kill.
- Review application memory usage.
- Increase the memory limit if justified by the workload.

### 2.4 Incorrect Command or Arguments

An incorrect startup command, invalid argument, or missing executable can cause the container to exit immediately.

**Example:**

```yaml
command: ["node"]
args: ["server-invalid.js"]
```

If `server-invalid.js` does not exist, the application will fail to start.

**Solution:**

- Verify the `command` and `args` fields in the Pod specification.
- Confirm that the required files and executables exist in the image.
- Review the container logs for startup errors.

### 2.5 Application Bugs and Exceptions

Unhandled exceptions, invalid application configuration, or runtime errors can cause the application to terminate.

**Example:**

A Node.js application crashes during startup because it cannot load a required module.

**Solution:**

- Inspect the application logs.
- Identify the exception or stack trace.
- Fix the application code or missing dependencies.
- Rebuild and redeploy the container image if required.

## 3. Step-by-Step Troubleshooting

Follow these commands in order when a Pod enters `CrashLoopBackOff`.

### Step 1: Check Pod Status

```bash
kubectl get pods -n <namespace>
```

Check the Pod's status and restart count.

### Step 2: Describe the Pod

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:
- Container state and last termination reason.
- Exit code.
- Liveness or startup probe failures.
- Events at the bottom of the output.
- `OOMKilled` status.

### Step 3: Check Current Container Logs

```bash
kubectl logs <pod-name> -n <namespace>
```

For a Pod with multiple containers:

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

### Step 4: Check Previous Container Logs

This is one of the most important commands because the current container may already have restarted.

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

For a specific container:

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace> --previous
```

### Step 5: Check Resource Usage

```bash
kubectl top pod <pod-name> -n <namespace>
```

This command requires Metrics Server to be available. It shows current usage, not historical usage or the exact memory peak before a crash.

### Step 6: Inspect the Deployment Configuration

```bash
kubectl get deployment <deployment-name> -n <namespace> -o yaml
```

Verify environment variables, image, command, arguments, probes, resource limits, and mounted volumes.

### Step 7: Apply the Fix and Verify

After correcting the root cause, monitor the Pod:

```bash
kubectl get pods -n <namespace> -w
```

Check the logs again:

```bash
kubectl logs <pod-name> -n <namespace>
```

Confirm that the application starts successfully and the restart count stops increasing.

## 4. Quick Troubleshooting Reference

| Symptom | What to check |
|---|---|
| Application exits immediately | Previous logs, command, arguments, exit code |
| `OOMKilled` | Memory usage and container memory limit |
| Liveness probe failures | Probe path, port, timing, application health |
| Database connection failure | Environment variables, Secrets, network connectivity |
| Missing configuration file | ConfigMap, volume mount, file path |
| Application exception | Logs, stack trace, code and dependencies |


## 5. Key Takeaway

**Always identify the root cause before restarting or redeploying the application.** Start with `kubectl describe pod` and `kubectl logs --previous`, then investigate the relevant configuration, resources, probes, or application code.
