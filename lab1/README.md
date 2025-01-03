## Task 1. Dockerfile

Write two Dockerfiles – a bad one and a good one. Also, write two bad practices for using containers.

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
### Description of the Dockerfile

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
- **FROM python:3.10** – sets the base image from which the new image will be built. In this case, the official Python image version 3.10 is used.

- **WORKDIR /app** – sets the working directory inside the container to /app.

- **COPY ./requirements.txt /app/server/requirements.txt \ ./alembic.ini /app/alembic.ini \ ./src /app/server/src** – copies several files from the build context (the directory where the Dockerfile is located) to the specified paths inside the container:
    - **requirements.txt** – the file listing the Python dependencies to be installed.
    - **alembic.ini** – the configuration file for Alembic (a tool for database migrations).
    - **src** – the directory containing the application’s source code.

- **RUN pip install --no-cache-dir --upgrade -r /app/server/requirements.txt** – installs the Python dependencies listed in requirements.txt using pip. The `--no-cache-dir` flag prevents the use of cached packages, and `--upgrade` updates packages to the latest version.
- **CMD ["uvicorn", "server.src.main:app", "--host", "0.0.0.0", "--port", "8000"]** – runs the Uvicorn server with the application defined in the main module from the `server.src` package, on the host `0.0.0.0` and port `8000`.
- **VOLUME /app/data** – all data written to this directory inside the container will be stored outside the container, allowing you to preserve it even after stopping or removing the container.


### Bad and Good Practices
1. **Using FROM python:latest in a Dockerfile is not recommended**:
    - The `latest` tag refers to the newest version of the base image at the time of the build. However, the contents of the image tagged `latest` can change over time as new versions are released. This means that when you rebuild the container later, you may get a completely different result because the base image has changed. This violates the principle of build reproducibility.
    - The `latest` tag does not specify the exact version of the base image, making it difficult to track dependencies and reproduce the environment in the future.
    - New versions of the base image may include changes that break compatibility with your application. Using `latest` can lead to unexpected failures or errors after the base image is updated.

2. **It is recommended to use apt-get update**:
    - Updating package lists: When you install a new package or update an existing one, your operating system must know where to look for these packages. `apt-get update` refreshes the list of available packages from repositories, ensuring your system has the most current information about available packages and their versions.
    - Preventing installation errors: If you do not run `apt-get update` before installing new packages, you may encounter errors due to outdated or unavailable package lists.
    - Ensuring security: Updating package lists is also important from a security standpoint. New updates may contain vulnerability fixes, and updating the package lists helps ensure you get the most secure versions of the packages.

3. **Not splitting RUN instructions into multiple lines**:
    - Increased image size: Each RUN instruction creates a new layer in the Docker image. It can slow down the build process and increase the image size.
    - Worse caching: When instructions are separated, Docker recreates all subsequent layers even if the input data did not change.

4. **Writing commands in lowercase**
    - Readability: Using a standard style where keywords begin with uppercase letters can improve the readability of the Dockerfile for other developers, especially if they are used to that style.


### When NOT to use containers at all
1. **Small projects or microservices with a low degree of isolation**:
    - If your project is very small or does not require complex infrastructure, containers can be excessive. For instance, if you have a simple application in a single programming language with no dependencies to isolate, installing it directly on the host machine may be simpler and less costly.

2. **Local development on separate machines**:
   - If your team works on separate machines and does not require a standardized development environment, containers may be unnecessary. Instead, each developer can configure their environment according to personal preferences.
