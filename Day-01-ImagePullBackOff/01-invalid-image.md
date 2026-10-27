# Invalid Image

`ImagePullBackOff` can occur when a Pod uses an invalid image name, contains a typo in the image reference, or specifies an image that does not exist in the container registry.

## Common Causes
- Incorrect image name.
- Typo in the image tag.
- Non-existent image in the registry.

## Example

```yaml
containers:
  - name: nginx
    image: nginxx:latest
```

Here, `nginxx:latest` is an incorrect image reference.

## Troubleshooting

```bash
kubectl describe pod <pod-name>
```

Check the **Events** section to identify the image-pull error.

## Solution

Correct the image name or tag:

```yaml
containers:
  - name: nginx
    image: nginx:latest
```

