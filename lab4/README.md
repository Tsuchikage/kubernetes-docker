## Task 4. More Kubernetes

Deploy your own service in Kubernetes, similar to Lab 3

**You can use Minikube from Lab 3. You need to deploy a service consisting of at least 2 containers + 1 init container, just like in Lab 2.
Requirements:**
- At least two Deployments, corresponding to the number of services
- A custom image for at least one of the Deployments (i.e., not public, built from your own Dockerfile)
- At least one Deployment must include both a container and an init container
- At least one Deployment must include a volume (any type)
- Mandatory use of a ConfigMap and/or Secret
- A Service is required for at least one of the services (which makes sense if they work in tandem)
- Liveness and/or Readiness probes in at least one of the Deployments
- Mandatory use of labels (in addition to the required selector/matchLabel, of course)

### Description
**Creating objects via CLI**
- Deploy your own service in Kubernetes, similar to Lab 3:
  - At least two Deployments, corresponding to the number of services
  - A custom image for at least one Deployment (i.e., not public, built from your own Dockerfile)
  - At least one Deployment must include both a container and an init container
  - At least one Deployment must include a volume (any)
  - Mandatory use of a ConfigMap and/or Secret
  - At least one Service is required for one of the services (which makes sense if they work in tandem)
  - At least one Deployment must have Liveness and/or Readiness probes
  - Mandatory use of labels (beyond the required selector/matchLabel)

- **configmap.yml**
  - Used to store configuration data that can be used by containers in a pod. In this case, it contains a single environment variable `APP_ENV` set to `production`.

- **Dockerfile**
  - Describes the process of creating a Docker image for a FastAPI application. Uses the Python 3.10 image, installs dependencies from `requirements.txt`, copies all the application files, and runs the application using Uvicorn.

- **fastapi-deployment-and-service.yml**
  - Deploys two replicas of the FastAPI application using a custom image. Includes an init container, uses a ConfigMap and a Secret, mounts a volume, and specifies a `livenessProbe`.
  - Creates a Service for FastAPI, allowing other applications to interact with it.

- **redis-deployment-and-service.yml**
  - Deploys a single replica of Redis.
  - Creates a Service for Redis, allowing other applications, such as FastAPI, to interact with Redis.

- **secret.yml**
  - Used to store confidential data. In this example, a secret key `SECRET_KEY` is stored.

- **main.py**
  - A simple application that connects to Redis and increments a counter with each request to the root URL (`/`).

___
### Run
```commandline
cd lab3
```

```commandline
minikube start
```
**Build the local image and load it into Minikube:**
- Used to configure the Windows (cmd) command-line environment to work with the Docker daemon that Minikube manages.
```commandline
@FOR /f "tokens=*" %i IN ('minikube docker-env --shell cmd') DO @%i
```
```commandline
docker build -t fastapi-app:local .
```
![image](/lab4/docs/1.png)
___
```commandline
kubectl create -f configmap.yml
kubectl create -f secret.yml
kubectl create -f fastapi-deployment-and-service.yml
kubectl create -f redis-deployment-and-service.yml
```
![image](/lab4/docs/2.png)
![image](/lab4/docs/4.png)
___
**OpenAPI:**
```commandline
minikube service fastapi-service --url
```
Example:
```commandline
http://127.0.0.1:58315/docs
```
![image](/lab4/docs/3.png)
___
```commandline
kubectl get pods
```
![image](/lab4/docs/1.png)
___
```commandline
kubectl get configmap
kubectl get deployment
kubectl get secret
kubectl get service
```
![image](/lab4/docs/5.png)
___
```commandline
kubectl describe pod <pod_name>
```

![image](/lab4/docs/6.png)

___
```commandline
kubectl config view
```
![image](/lab4/docs/7.png)
