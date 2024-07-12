# Building Docker Images in Kubernetes Pods using Kaniko

## What we will learn

1. **Build images with and without Docker Daemon**
2. **Understanding Docker-in-Docker (DIND)**
3. **Use of /var/run/docker.sock file in Docker**
4. **Making use of Kaniko in Jenkins as an agent to build images**


### 1. Build images with and without Docker Daemon

**1.1 Build images with Docker Daemon**

![Build images with Docker Daemon](https://miro.medium.com/v2/resize:fit:1400/1*mMwScMsMUafCPs9iwP060Q.png)

#### Sample Dockerfile for a Flask Application

```Dockerfile
# Use the official Python image from the Docker Hub
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the required dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code into the container
COPY . .

# Expose the port the app runs on
EXPOSE 5000

# Define the command to run the application
CMD ["python", "app.py"]
```
#### Docker Build Command
      To build the Docker image for the Flask application, use the following command:
      
```sh
      docker build -t my-flask-app .
```

**1.2  Why Use `/var/run/docker.sock`?**

- **Direct Communication**: By mounting the Docker socket into a container, the container can communicate directly with the Docker daemon of the host system. This allows the container to run Docker commands as if they were executed on the host.

- **Container Management**: This setup is often used in CI/CD pipelines and other automation tools where containers need to build, start, stop, or manage other containers dynamically.

#### Security Considerations

- **Access Control**: Mounting the Docker socket into a container grants the container full control over the Docker daemon and, by extension, the entire host system. This can pose significant security risks if not managed properly.

- **Best Practices**: Ensure that only trusted containers and applications have access to the Docker socket. Consider using alternative methods, such as using tools like Kaniko, which do not require Docker daemon access.

#### Example Usage

Here’s an example of how you might mount the Docker socket into a container:

docker run -v /var/run/docker.sock:/var/run/docker.sock -it your-docker-image

**1.3  Kubernetes removed Docker from its core**

## Why Kubernetes Removed Docker from Its Core?

**Container Runtime Interface (CRI):** Kubernetes introduced the Container Runtime Interface (CRI) to standardize how container runtimes interface with Kubernetes. Initially, Docker was the default runtime using the `dockerd` service and Docker API (`docker.sock`). However, Kubernetes started encouraging the use of CRI-compliant runtimes to support a broader ecosystem of container engines.

**Decoupling from Docker:** Kubernetes removed direct dependencies on Docker as its default runtime due to several reasons:
- **Flexibility:** Supporting multiple runtimes allows Kubernetes to cater to different use cases and environments beyond Docker.
- **Standardization:** CRI allows Kubernetes to maintain a stable API for integrating with various container runtimes, not limited to Docker.
- **Ecosystem Support:** Encourages innovation and integration of new container runtimes like containerd, CRI-O, and others.

**Impact on `docker.sock` Usage:** With Docker no longer being the primary runtime in Kubernetes, using `/var/run/docker.sock` to directly communicate with Docker is not compatible with CRI-based runtimes. Kubernetes nodes typically run a CRI implementation (e.g., containerd, CRI-O), which Kubernetes components interact with instead.

## Alternatives to Using Docker in Kubernetes

- **containerd:** Developed by Docker, containerd is a lightweight container runtime that implements CRI and is designed for use with Kubernetes. It's widely adopted and maintained by the CNCF.

- **CRI-O:** An implementation of the Kubernetes CRI to provide an integration path between OCI conformant runtimes like runc (the underlying runtime used by containerd) and Kubernetes. It's optimized for Kubernetes and supports the CRI standards.

- **Others:** Kubernetes supports various other CRI-compliant runtimes like frakti (for hypervisor-based environments), Kata Containers (for secure, lightweight VM-based containers), and more.

## Transitioning Away from `docker.sock` Usage in Kubernetes

- **Update Kubernetes Manifests:** Modify Kubernetes deployment manifests (yaml files) to specify the correct container runtime using CRI-compatible configurations (`containerRuntime: remote` or similar in `kubelet` configuration).

- **Tooling and Automation:** Update CI/CD pipelines, automation scripts, and tools (e.g., Helm charts) to reflect the use of CRI-compliant runtimes instead of Docker-specific configurations.

- **Best Practices:** Ensure that Kubernetes configurations align with recommended practices from Kubernetes documentation and the chosen container runtime provider (e.g., containerd, CRI-O).


    

