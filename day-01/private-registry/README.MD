
# Creating a Docker Image for a Custom Python Application, Pushing it to Docker Hub, and Deploying with Kubernetes

In this guide, you will learn step-by-step how to create a Docker image for a custom Python application, push it to a private Docker Hub repository, and deploy it with Kubernetes.

## Python Application

Let’s start by creating a Python-based application. Here’s a simple example of a Flask application:

**app.py**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

##### Python Application Dependencies
The libraries required for your Python application to work:

**requirements.txt**

```
Flask==2.1.1
Werkzeug==2.0.2
```

##### Building the Docker Image
Now, let’s use a Dockerfile to create a Docker image:

**Dockerfile**
```
FROM python:3.8
RUN mkdir /app
COPY app.py /app
COPY requirements.txt /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

You can build the Docker image and run it locally using the following commands:

```
docker build -t flask-app .
docker run -p 5000:5000 flask-app
```

##### Pushing to Docker Hub
To push the image to Docker Hub, follow these steps:

```
docker tag flask-app:latest your-dockerhub-username/flask-app:1.0
docker login
docker push your-dockerhub-username/flask-app:1.0
```

##### Deployment on Kubernetes
Now, let’s configure the necessary secrets to access the Docker Hub image when deploying on Kubernetes:

```
kubectl create secret docker-registry docker-cred \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=your-username \
    --docker-password=your-password
```

Once the secrets are set up, proceed by applying the changes to the Deployment YAML file. We’ve added the imagePullSecrets field to the Deployment YAML file and set its value to the name of the secret docker-cred. Kubernetes will use this secret for authentication when pulling the your-dockerhub-username/flask-app:1.0 image.

After setting up the secrets, you can proceed with deploying your application on Kubernetes using the provided Deployment and Service YAML files.

**deployment.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: flask
  name: flask
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - image: your-dockerhub-username/flask-app:1.0
        imagePullPolicy: Always
        name: flask
      imagePullSecrets:
      - name: docker-cred
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: flask
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30007
```

Apply the Deployment and Service configuration:

```
kubectl apply -f deployment.yaml
kubectl get deploy
kubectl get po -o wide
kubectl get svc -o wide
kubectl get nodes -o wide
```

Another way to provide Docker registry credentials to a Kubernetes deployment is by creating and mounting a service account with the necessary image pull secret. This approach avoids adding imagePullSecrets to each deployment manually.

Here's a step-by-step example of how to do this:

**Create a Docker Registry Secret**

First, create the Docker registry secret:

bash
Copy code
```kubectl create secret docker-registry docker-cred \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=your-username \
    --docker-password=your-password
```
Create a Service Account with the Image Pull Secret

Create a YAML file to define a service account that uses this secret:

service-account.yaml

yaml
Copy code
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
secrets:
- name: docker-cred
```
Apply the service account configuration:

bash
Copy code
```
kubectl apply -f service-account.yaml
```
Create the Deployment Using the Service Account

Define the deployment to use the service account created above:

**deployment.yaml**

yaml
Copy code
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: flask
  name: flask
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      serviceAccountName: my-service-account
      containers:
      - image: your-dockerhub-username/flask-app:1.0
        imagePullPolicy: Always
        name: flask
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: flask
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30007
```
Apply the deployment configuration:

bash
Copy code
```
kubectl apply -f deployment.yaml
```
Verify the Deployment

Ensure the deployment and services are running correctly:

bash
Copy code
```
kubectl get deploy
kubectl get po -o wide
kubectl get svc -o wide
kubectl get nodes -o wide
```
By using a service account with an image pull secret, you avoid the need to specify imagePullSecrets in each deployment specification, making your configurations cleaner and easier to manage.

