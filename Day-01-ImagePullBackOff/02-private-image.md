# Private Image

A Pod may encounter `ImagePullBackOff` when Kubernetes cannot authenticate with a private container image registry.

To resolve this, create a Kubernetes Secret containing registry credentials and reference it using `imagePullSecrets`.

## 1. Create a Secret Using Docker Credentials

```bash
kubectl create secret docker-registry demo \
  --docker-server=<your-registry-server> \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email> \
  --namespace=default
```

Replace the placeholders with your registry details.

## 2. Create a Secret Using AWS ECR Credentials

```bash
kubectl create secret docker-registry ecr-secret \
  --docker-server=${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com \
  --docker-username=AWS \
  --docker-password="$(aws ecr get-login-password --region ${AWS_REGION})" \
  --namespace=default
```

**Prerequisites:**
- AWS CLI configured with appropriate ECR permissions.
- Correct AWS account ID and region.
- Valid ECR repository and image.

Note: ECR authorization tokens expire after 12 hours. For EKS, worker-node IAM permissions are generally preferred over manually created pull Secrets for standard ECR image pulls.

## 3. Use the Secret in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  containers:
    - name: app
      image: <your-private-image>
  imagePullSecrets:
    - name: demo
```

For AWS ECR, replace `demo` with `ecr-secret` if you created that Secret.

## 4. Apply and Verify

```bash
kubectl apply -f private-pod.yaml
kubectl get pods
kubectl describe pod private-app
```

**Key takeaway:** The Secret must exist in the same namespace as the Pod and contain valid registry credentials.

