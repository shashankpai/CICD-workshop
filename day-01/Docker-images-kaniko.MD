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

#### Why Kubernetes Removed Docker from Its Core?

**Container Runtime Interface (CRI):** Kubernetes introduced the Container Runtime Interface (CRI) to standardize how container runtimes interface with Kubernetes. Initially, Docker was the default runtime using the `dockerd` service and Docker API (`docker.sock`). However, Kubernetes started encouraging the use of CRI-compliant runtimes to support a broader ecosystem of container engines.

**Decoupling from Docker:** Kubernetes removed direct dependencies on Docker as its default runtime due to several reasons:
- **Flexibility:** Supporting multiple runtimes allows Kubernetes to cater to different use cases and environments beyond Docker.
- **Standardization:** CRI allows Kubernetes to maintain a stable API for integrating with various container runtimes, not limited to Docker.
- **Ecosystem Support:** Encourages innovation and integration of new container runtimes like containerd, CRI-O, and others.

**Impact on `docker.sock` Usage:** With Docker no longer being the primary runtime in Kubernetes, using `/var/run/docker.sock` to directly communicate with Docker is not compatible with CRI-based runtimes. Kubernetes nodes typically run a CRI implementation (e.g., containerd, CRI-O), which Kubernetes components interact with instead.

#### Alternatives to Using Docker in Kubernetes

- **containerd:** Developed by Docker, containerd is a lightweight container runtime that implements CRI and is designed for use with Kubernetes. It's widely adopted and maintained by the CNCF.

- **CRI-O:** An implementation of the Kubernetes CRI to provide an integration path between OCI conformant runtimes like runc (the underlying runtime used by containerd) and Kubernetes. It's optimized for Kubernetes and supports the CRI standards.

- **Others:** Kubernetes supports various other CRI-compliant runtimes like frakti (for hypervisor-based environments), Kata Containers (for secure, lightweight VM-based containers), and more.

** Kaniko - What Problem kaniko is addressing **

#### Containerized Build Process
- Kaniko runs as a container image itself. When initiated, it creates a pod (in Kubernetes terms) or a container that executes the build process.

#### No Docker Daemon Dependency
- Kaniko doesn't require a Docker daemon (`dockerd`). Instead, it interacts directly with container registries and the filesystem to execute Dockerfile commands.

#### Layered Image Building
- Similar to Docker builds, Kaniko processes each instruction in the Dockerfile to create layers of the container image.

#### Secure Execution
- Kaniko ensures security by running builds in an isolated environment without needing privileged access or a Docker daemon socket (`/var/run/docker.sock`).

#### Technologies Used by Kaniko
- **OCI (Open Container Initiative) Specifications:** Kaniko adheres to OCI standards for container images and uses these specifications for compatibility and interoperability with various container runtimes and registries.
- **Google Container Tools:** Kaniko is developed and maintained by Google, leveraging their expertise in containerization and cloud-native technologies.
- **Build Context:** Kaniko uses the provided build context (the directory containing the Dockerfile and related files) to execute build commands and create the container image.

### Advantages of Using Kaniko
- **Security:** Eliminates the need for privileged Docker daemon access, reducing potential attack surfaces and enhancing build pipeline security.
- **Portability:** Kaniko can be used in various Kubernetes environments and other container orchestration platforms that support containerized builds.
- **Compatibility:** Works with Dockerfiles and Dockerfile syntax, allowing developers to use familiar build processes without requiring changes.

In summary, Kaniko uses containerization techniques to provide a secure and efficient method for building container images without relying on a Docker daemon. This makes it particularly useful in Kubernetes and other cloud-native environments where security and portability are critical considerations.


