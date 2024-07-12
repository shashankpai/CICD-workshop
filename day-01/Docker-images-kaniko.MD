# Building Docker Images in Kubernetes Pods using Kaniko

## What we will learn

1. **Build images with and without Docker Daemon**
2. **Understanding Docker-in-Docker (DIND)**
3. **Use of /var/run/docker.sock file in Docker**
4. **Making use of Kaniko in Jenkins as an agent to build images**


1. **Build images with and without Docker Daemon**

1.1 **Build images with Docker Daemon**

![Build images with Docker Daemon](https://www.google.com/url?sa=i&url=https%3A%2F%2Fmedium.com%2Fgeekculture%2Fcreate-docker-images-without-docker-daemon-kaniko-847a688155a6&psig=AOvVaw1O4Q1C6JwpbPAGUiHYwCuw&ust=1720860586773000&source=images&cd=vfe&opi=89978449&ved=0CBEQjRxqFwoTCJiHjrWPoYcDFQAAAAAdAAAAABAE)
    - ### Sample Dockerfile for a Flask Application
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

    - ### Docker Build Command
      To build the Docker image for the Flask application, use the following command:
      ```sh
      docker build -t my-flask-app .
      ```
