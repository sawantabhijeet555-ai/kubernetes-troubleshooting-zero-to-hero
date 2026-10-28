# Docker Setup and Deployment Guide

## Building and Running Your Application

When you're ready to build and run your application locally, execute the following command:

```bash
docker compose up --build
```

This command builds the Docker images (if required) and starts the application services defined in your `compose.yaml` or `docker-compose.yml` file.

Once the application starts successfully, access it at:

**Application URL:** http://localhost:8000

To stop the running containers, press `Ctrl+C`. You can also stop the services using:

```bash
docker compose down
```

## Deploying Your Application to the Cloud

Follow these steps to build your Docker image, push it to a container registry, and prepare it for cloud deployment.

### Step 1: Build the Docker Image

Run the following command from the directory containing your `Dockerfile`:

```bash
docker build -t myapp .
```

Replace `myapp` with your application's image name.

### Step 2: Build for a Specific Platform (Optional)

If your development machine and cloud environment use different CPU architectures, specify the target platform during the build.

For example, to build an image for Linux AMD64:

```bash
docker build --platform=linux/amd64 -t myapp .
```

This is useful when building on an ARM-based machine, such as an Apple Silicon Mac, for deployment to an AMD64 cloud environment.

### Step 3: Tag the Docker Image

Tag your image with the address of your container registry:

```bash
docker tag myapp myregistry.com/myapp:latest
```

Replace `myregistry.com/myapp` with your actual registry address and repository name.

### Step 4: Authenticate with the Container Registry

Log in to your registry using the appropriate authentication method.

For a registry that supports Docker login:

```bash
docker login myregistry.com
```

Follow the registry's authentication instructions. Avoid hardcoding passwords or access tokens in scripts.

### Step 5: Push the Docker Image

Upload the image to your container registry:

```bash
docker push myregistry.com/myapp:latest
```

After the push completes, your cloud environment can pull the image from the registry, provided it has the necessary access permissions.

### Step 6: Deploy to Your Cloud Environment

Configure your cloud platform to use the pushed image.

For example, in Kubernetes, reference the image in your Deployment manifest:

```yaml
containers:
  - name: myapp
    image: myregistry.com/myapp:latest
    ports:
      - containerPort: 8000
```

Ensure that the image is accessible to your cluster and that the application's ports, environment variables, resource requirements, and networking are configured correctly.

## References

- [Docker's Python Guide](https://docs.docker.com/language/python/)
- [Docker: Building and Sharing Your Image](https://docs.docker.com/go/get-started-sharing/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Build Documentation](https://docs.docker.com/build/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
