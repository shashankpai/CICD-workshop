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

**1.4 Kaniko - What Problem kaniko is addressing**

##### Containerized Build Process
- Kaniko runs as a container image itself. When initiated, it creates a pod (in Kubernetes terms) or a container that executes the build process.

##### No Docker Daemon Dependency
- Kaniko doesn't require a Docker daemon (`dockerd`). Instead, it interacts directly with container registries and the filesystem to execute Dockerfile commands.

##### Layered Image Building
- Similar to Docker builds, Kaniko processes each instruction in the Dockerfile to create layers of the container image.

##### Secure Execution
- Kaniko ensures security by running builds in an isolated environment without needing privileged access or a Docker daemon socket (`/var/run/docker.sock`).

##### Technologies Used by Kaniko
- **OCI (Open Container Initiative) Specifications:** Kaniko adheres to OCI standards for container images and uses these specifications for compatibility and interoperability with various container runtimes and registries.
- **Google Container Tools:** Kaniko is developed and maintained by Google, leveraging their expertise in containerization and cloud-native technologies.
- **Build Context:** Kaniko uses the provided build context (the directory containing the Dockerfile and related files) to execute build commands and create the container image.

##### Advantages of Using Kaniko
- **Security:** Eliminates the need for privileged Docker daemon access, reducing potential attack surfaces and enhancing build pipeline security.
- **Portability:** Kaniko can be used in various Kubernetes environments and other container orchestration platforms that support containerized builds.
- **Compatibility:** Works with Dockerfiles and Dockerfile syntax, allowing developers to use familiar build processes without requiring changes.

In summary, Kaniko uses containerization techniques to provide a secure and efficient method for building container images without relying on a Docker daemon. This makes it particularly useful in Kubernetes and other cloud-native environments where security and portability are critical considerations.

## 1.5 How Kaniko Builds Images

**Kaniko** is designed to build container images in a secure and efficient manner, especially in environments like Kubernetes where direct access to a Docker daemon (`dockerd`) is not feasible or desired.

##### Kaniko Executor Image:

- Kaniko itself runs as a container image known as the Kaniko executor image (`gcr.io/kaniko-project/executor`). This image contains the necessary tools and configurations to execute Dockerfile instructions and build container images.
- It is lightweight and designed to operate within Kubernetes or other container orchestration platforms, independent of a Docker daemon.

##### Building Process with Kaniko:

- **Containerized Execution**: Kaniko creates a pod (in Kubernetes terminology) or a container to perform the build process. This pod/container executes the commands specified in the Dockerfile to build the desired container image.
- **Context and Dockerfile**: Kaniko requires a build context, typically a directory containing the Dockerfile and any necessary files (`context.tar.gz`). It uses this context to execute commands like `COPY`, `RUN`, and `ENTRYPOINT` defined in the Dockerfile.
- **Layered Image Construction**: Similar to Docker, Kaniko builds images in layers based on the instructions in the Dockerfile, optimizing caching and reusability of intermediate layers.

##### Kaniko Executor Image Arguments:

- When using Kaniko, the executor image expects specific arguments to configure the build process. These commonly include:
  - `--context`: Path to the build context (e.g., `/workspace/context`).
  - `--dockerfile`: Path to the Dockerfile within the context (e.g., `/workspace/context/Dockerfile`).
  - Optionally, other flags like `--destination` to specify the final image repository and tag.

##### Example of a Pod Using Kaniko

Here's an example YAML configuration for a Kubernetes pod that uses Kaniko to build a container image:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kaniko-build-pod
spec:
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      args: ["--context", "/workspace/context", "--dockerfile", "/workspace/context/Dockerfile"]
      volumeMounts:
        - name: kaniko-workspace
          mountPath: /workspace/context
  volumes:
    - name: kaniko-workspace
      emptyDir: {}
```
In this example:

The kaniko container runs the Kaniko executor image `gcr.io/kaniko-project/executor:latest`.
It specifies arguments (--context and --dockerfile) to indicate where the build context and Dockerfile are located.
A volume `kaniko-workspace` is mounted to provide the build context `/workspace/context` to Kaniko.
Replace /workspace/context and /workspace/context/Dockerfile with your actual paths and filenames as per your project structure.

#### Example of building and pushing to a docker registry using kaniko in a pod


##### First create a kubernetes secret with docker registry credentials
```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

```
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=shapai --docker-password=<your-pword> --docker-email=<your-email>
```


