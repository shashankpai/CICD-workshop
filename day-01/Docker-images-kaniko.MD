# Building Docker Images in Kubernetes Pods using Kaniko

## What we will learn

1. **Build images with and without Docker Daemon**
2. **Understanding Docker-in-Docker (DIND)**
3. **Use of /var/run/docker.sock file in Docker**
4. **Making use of Kaniko in Jenkins as an agent to build images**


### 1. Build images with and without Docker Daemon

1.1 **Build images with Docker Daemon**

![Build images with Docker Daemon](https://miro.medium.com/v2/resize:fit:1400/1*mMwScMsMUafCPs9iwP060Q.png)

### Sample Dockerfile for a Flask Application

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


1.2  **Why Use `/var/run/docker.sock`?**

- **Direct Communication**: By mounting the Docker socket into a container, the container can communicate directly with the Docker daemon of the host system. This allows the container to run Docker commands as if they were executed on the host.

- **Container Management**: This setup is often used in CI/CD pipelines and other automation tools where containers need to build, start, stop, or manage other containers dynamically.

### Security Considerations

- **Access Control**: Mounting the Docker socket into a container grants the container full control over the Docker daemon and, by extension, the entire host system. This can pose significant security risks if not managed properly.

- **Best Practices**: Ensure that only trusted containers and applications have access to the Docker socket. Consider using alternative methods, such as using tools like Kaniko, which do not require Docker daemon access.

### Example Usage

Here’s an example of how you might mount the Docker socket into a container:

docker run -v /var/run/docker.sock:/var/run/docker.sock -it your-docker-image

### Docker-in-Docker (DIND) vs. Mounting Docker Socket

- **DIND**: Useful for completely isolated Docker environments where the container itself runs a separate Docker daemon.
  
  ```sh
  docker run --privileged -d --name dind-container docker:dind
  ```



    

