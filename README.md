# Containerization and application orchestration, ITMO, Spring 2024

## Description
This course is dedicated to familiarizing and working with containerization technologies and DevOps practices. We propose to take a detailed look at containerization and orchestration technologies, their features, advantages/disadvantages, and whether they are truly universally necessary and why. Special attention will be paid to the nature of the DevOps concept, as well as the features of microservice infrastructure. In addition, as part of the discipline, practical skills in working with Docker, Docker Compose, and Kubernetes will be acquired.


## Contents of the laboratory assignments

## Task 1. Dockerfile

Write two Dockerfiles – a bad one and a good one. Write two bad practices for using containers.

### Run
```commandline
cd lab1\server
```
```commandline
docker build -t test_name .
```
```commandline
docker run -d --name test_name -p 8000:8000 test_name
```
```commandline
OpenAPI: http://localhost:8000/api/docs
```
### Description of Dockerfile

**Bad Dockerfile**
```
from python:latest

workdir /app

copy ./requirements.txt /app/server/requirements.txt
copy ./alembic.ini /app/alembic.ini
copy ./src /app/server/src

run pip install --no-cache-dir --upgrade -r /app/server/requirements.txt

cmd ["uvicorn", "server.src.main:app", "--host", "0.0.0.0", "--port", "8000"]

volume /app/data
```


**Good Dockerfile**
```
FROM python:3.10

WORKDIR /app

COPY ./requirements.txt /app/server/requirements.txt
COPY ./alembic.ini /app/alembic.ini
COPY ./src /app/server/src

RUN apt-get update && pip install --no-cache-dir --upgrade -r /app/server/requirements.txt

CMD ["uvicorn", "server.src.main:app", "--host", "0.0.0.0", "--port", "8000"]

VOLUME /app/data
```
Creates a Docker image based on Python 3.10.
- **FROM python:3.10** - sets the base image on which the new image will be built. In this case, the official Python image version 3.10 is used.

- **WORKDIR /app** - sets the working directory inside the container to /app.

- **COPY ./requirements.txt /app/server/requirements.txt \ ./alembic.ini /app/alembic.ini \ ./src /app/server/src** - copies several files from the build context (the directory where Dockerfile is located) to the specified paths inside the container:
    - **requirements.txt** - a file with a list of Python dependencies to install.
    - **alembic.ini** - the configuration file for Alembic (a tool for database migrations).
    - **src** - the directory containing the application source code.

- **RUN pip install --no-cache-dir --upgrade -r /app/server/requirements.txt** - installs the Python dependencies listed in requirements.txt using pip. The --no-cache-dir flag prevents the use of package caches, and --upgrade updates packages to the latest version.
- **CMD ["uvicorn", "server.src.main:app", "--host", "0.0.0.0", "--port", "8000"]** - runs the uvicorn server with the application defined in the main module from the server.src package, on the host "0.0.0.0" and port "8000".
- **VOLUME /app/data** - all data written to this directory inside the container will be stored outside the container, which allows you to keep it even after the container is stopped or deleted.


### Bad and good practices
1. **Using FROM python:latest in Dockerfile is not recommended**:
    - The latest tag points to the freshest version of the base image at the time of the build. However, the contents of the image with the latest tag can change over time as new versions are released. This means that when you rebuild the container later, you may get a completely different result because the base image has changed. This violates the principle of build reproducibility.
    - The latest tag does not specify the exact version of the base image, which complicates tracking dependencies and reproducing the environment in the future.
    - New versions of the base image may contain changes that break compatibility with your application. Using latest can lead to unforeseen failures or errors after the base image is updated.

2. **It is recommended to use apt-get update**:
    - Updating package lists: When installing or updating packages, your operating system should know where to find them. apt-get update updates the list of available packages from the repositories so that your system has the most up-to-date information on available packages and their versions.
    - Preventing installation errors: If you do not run apt-get update before installing new packages, you may run into errors due to outdated or unavailable package lists.
    - Ensuring security: Updating the package list is also important from a security standpoint. New updates may contain vulnerability fixes, and updating the package lists helps ensure you get the latest secure versions of the packages.

3. **Not using multiple RUN instructions (breaking them into multiple lines)**:
    - Increased image size: Each RUN instruction creates a new layer in a Docker image. It can slow down the build process and increase its size.
    - Worse caching: When instructions are separated, Docker recreates all subsequent layers, even if the input data did not change.

4. **Using lowercase commands**
    - Readability: Using the standard style where keywords start with an uppercase letter can improve the readability of a Dockerfile for other developers, especially if they are used to this style.


### When NOT to use containers at all
1. **Small projects or microservices with a low degree of isolation**:
    - If your project is very small or does not require complex infrastructure, containers can be redundant. For example, if you have a simple application written in one programming language with no dependencies that need isolation, installing it directly on the host machine may be simpler and less resource-intensive.

2. **Local development on individual machines**:
   - If your team works on separate machines and does not need a standardized development environment, containers can be excessive. Instead, each developer can configure their environment according to their own preferences.


## Task 2. Docker Compose

Create a docker-compose.yml with at least three services

**Based on the Dockerfile from Lab 1, create a Compose project. Mandatory requirements:**
- at least 1 init + 2 app services (one-time init + application + database or something else, as long as they work in tandem)
- automatic image build from the Dockerfile located nearby and assigning it (the image) a name
- strict naming of the resulting containers
- at least one of the services must have depends_on
- at least one of the services must have a volume
- at least one of the services must expose a port to the outside
- at least one of the services must have a command and/or entrypoint key (you can reuse the one from the Dockerfile)
- add a healthcheck
- all envs must be specified not in the docker-compose.yml itself, but in a .env file located nearby
- a network must be explicitly specified (one for all)

### Run
```commandline
cd lab2
```
```commandline
cp .env.example .env
```
```commandline
docker-compose up
```
OpenAPI:
```commandline
http://localhost:8000/api/docs
```
Health Check:
```commandline
http://localhost:8000/health
```

### Answers to questions
**1. Is it possible to limit resources (e.g., memory or CPU) for services in docker-compose.yml? If not, why not; if yes, how?**
Yes, in Docker Compose, you can limit resources for containers using special parameters in the docker-compose.yml file.

To limit memory for a container, you can use the mem_limit parameter.

**For example:**
```yaml
services:
  my_service:
    image: my_image
    memory: 512m # Sets the memory limit to 512 MB
```
To limit CPU usage, you can use the cpu_quota and cpu_period parameters. cpu_quota defines the share of CPU available to the container, and cpu_period sets the period of time over which this quota is applied.

**For example:**
```yaml
services:
  my_service:
    image: my_image
    cpu_quota: 50000 # Limits CPU usage to 50% of one core
    cpu_period: 100000 # The time period in microseconds (100000 = 100 ms)
```
In this example, the my_service container is limited to using 50% of one CPU core.

Resource constraints help prevent a situation where one container consumes too many resources and affects the operation of other containers or the Docker host.


\
**2. How can you run only a specific service from docker-compose.yml without running the others?**

To run only a specific service from docker-compose.yml, without running the rest, you can use the command:
```commandline
docker-compose up service_name
```
Replace <service_name> with the name of the service you want to run.


## Task 3. Kubernetes

Install Kubernetes on your local machine (the example is for minikube on Windows 10/11). Deploy a test service.

**Steps:**
- Install minikube
- Create objects via CLI
- Connect from the outside

**And also make manipulations on the manifests from the example to achieve the following:**
- For Postgres, move POSTGRES_USER and POSTGRES_PASSWORD from the ConfigMap to Secrets (obviously, a new manifest for the Secret entity is needed)
- For Nextcloud, move its variables (NEXTCLOUD_UPDATE, ALLOW_EMPTY_PASSWORD, etc.) from the Deployment to the ConfigMap (obviously, a new manifest for the ConfigMap entity is needed)
- For Nextcloud, add Liveness and Readiness probes

### Run
```commandline
cd lab3
```
___
**Creating objects via CLI**
- Start minikube and create the yml files (manifests) for the ConfigMap, Service, and Deployment.
  - Make manipulations on the manifests:
      - For Postgres, move POSTGRES_USER and POSTGRES_PASSWORD from the ConfigMap to Secrets (obviously, a new manifest for the Secret entity is needed)
      - For Nextcloud, move its variables (NEXTCLOUD_UPDATE, ALLOW_EMPTY_PASSWORD, etc.) from the Deployment to the ConfigMap (obviously, a new manifest for the ConfigMap entity is needed)
      - For Nextcloud, add Liveness and Readiness probes
```commandline
minikube start
kubectl create -f pg_configmap.yml
kubectl create -f postgres-secrets.yml
kubectl create -f pg_service.yml
kubectl create -f pg_deployment.yml
kubectl create -f nextcloud_configmap.yml
kubectl create -f nextcloud.yml
```

```commandline
kubectl get pods
```
![image](/lab3/docs/1.png)
___
```commandline
kubectl get configmap
kubectl get deployment
kubectl get secret
kubectl get service
```
![image](/lab3/docs/4.png)
___
```commandline
kubectl describe pod <pod_name>
```

![image](/lab3/docs/2.png)

___
```commandline
kubectl config view
```
![image](/lab3/docs/3.png)
___

### Questions
**Is the order of execution of these manifests important? Why?**

Yes, the order of these manifests is important for the following reasons:
- **pg_configmap.yml** and **postgres-secrets.yml** create ConfigMap and Secret needed for the configuration and launch of PostgreSQL. These resources must be created before the PostgreSQL deployment is created, as the deployment uses these resources to configure the container environment.
- **pg_service.yml** creates the service that provides network access to the PostgreSQL database inside Kubernetes. This is important so that Nextcloud can connect to the database by the name of the service 'postgres-service'.
- **pg_deployment.yml** creates the PostgreSQL deployment, which depends on the existing ConfigMap and Secret to run successfully.
- **nextcloud_configmap.yml** creates a ConfigMap for Nextcloud, containing the settings needed for Nextcloud initialization and configuration.
- **nextcloud.yml** creates the Nextcloud deployment, which uses the ConfigMap and Secret for configuration, and also connects to the PostgreSQL database via the postgres-service.

**What (and why) will happen if you scale the number of replicas of postgres-deployment to 0, then back to 1, and then try to log in to Nextcloud again?**
- When stopping and restarting the PostgreSQL pod, the database data is not saved. When you try to log in to Nextcloud after restarting the PostgreSQL pod, Nextcloud will not be able to connect to the database because the data was lost.
- To ensure data persistence between pod restarts, you need to use PersistentVolume and PersistentVolumeClaim to store the database data outside the pod.

- ![image](/lab3/docs/5.png)
- ![image](/lab3/docs/6.png)


## Task 4. More Kubernetes

### Task
Deploy your own service in Kubernetes, similar to Lab 3

**You can use Minikube from Lab 3. You need to deploy a service in a bundle of at least 2 containers + 1 init, similar to Lab 2. 
Requirements:**
- at least two Deployments, by the number of services
- a custom image for at least one Deployment (i.e., not public and built from your own Dockerfile)
- at least one Deployment should contain a container and an init-container
- at least one Deployment should contain a volume (any)
- mandatory use of ConfigMap and/or Secret
- mandatory Service for at least one of the services (which makes sense if they work in tandem)
- Liveness and/or Readiness probes in at least one of the Deployments
- mandatory use of labels (in addition to the required selector/matchLabel, of course)

### Description
**Creating objects via CLI**
- Deploying our own service in Kubernetes, similar to Lab 3
  - at least two Deployments, by the number of services
  - a custom image for at least one Deployment (i.e., not public and built from your own Dockerfile)
  - at least one Deployment should contain a container and an init-container
  - at least one Deployment should contain a volume (any)
  - mandatory use of ConfigMap and/or Secret
  - mandatory Service for at least one of the services (which makes sense if they work in tandem)
  - Liveness and/or Readiness probes in at least one of the Deployments
  - mandatory use of labels (in addition to the required selector/matchLabel, of course)


- **configmap.yml**
  - Used to store configuration data that can be used by containers in a pod. In this case, there is one environment variable APP_ENV set to production.
- **Dockerfile**
  - Describes the process of creating a Docker image for a FastAPI application. Uses the Python 3.10 image, installs dependencies from requirements.txt, copies all application files, and launches the app via Uvicorn.
- **fastapi-deployment-and-service.yml**
  - Deploys two replicas of the FastAPI application using a custom image. Includes an init-container, uses ConfigMap and Secret, mounts a volume, and defines a livenessProbe.
  - Creates a service for FastAPI that allows other applications to interact with it.
- **redis-deployment-and-service.yml**
  - Deploys one replica of Redis.
  - Creates a service for Redis, allowing other applications such as FastAPI to interact with Redis.
- **secret.yml**
  - Used to store confidential data. In this case, a SECRET_KEY is stored.
- **main.py**
  - A simple application that connects to Redis and increments a counter with each request to the root URL (/).

___
### Run
```commandline
cd lab3
```
  
```commandline
minikube start
```
**Build the local image and load it into Minikube:**
- Used to configure the Windows (cmd) command-line environment to work with Docker managed by Minikube.
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
___
