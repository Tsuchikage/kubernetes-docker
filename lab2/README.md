## Task 2. Docker Compose

Create a `docker-compose.yml` file with at least three services

**Based on the Dockerfile from Lab 1, create a Compose project. Mandatory requirements:**
- At least 1 init + 2 app services (a one-time init + an application + a database or something else, as long as they work together)
- Automatic image build from the Dockerfile located nearby, and assigning a name to that image
- Strict naming for the resulting containers
- At least one of the services must have `depends_on`
- At least one of the services must have a volume
- At least one of the services must expose a port to the outside
- At least one of the services must use the `command` and/or `entrypoint` key (you can reuse the one from the Dockerfile)
- Add a healthcheck
- All environment variables (`env`s) must be specified not in the `docker-compose.yml` file itself, but in a nearby `.env` file
- A network must be explicitly specified (one for all)

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
**1. Is it possible to limit resources (e.g., memory or CPU) for services in `docker-compose.yml`? If not, why not? If yes, how?**  
Yes, in Docker Compose you can limit resources for containers by using special parameters in the `docker-compose.yml` file.  
To limit a container’s memory, you can use the `mem_limit` parameter.

**Example:**
```yaml
services:
  my_service:
    image: my_image
    memory: 512m # Sets a memory limit of 512 MB
```
To limit CPU usage, you can use the `cpu_quota` and `cpu_period` parameters. `cpu_quota` specifies the share of CPU available to the container, and `cpu_period` sets the time period over which this quota is applied.

**Example:**
```yaml
services:
  my_service:
    image: my_image
    cpu_quota: 50000  # Limits CPU usage to 50% of one CPU core
    cpu_period: 100000  # The time period in microseconds (100000 = 100 ms)
```
In this example, the `my_service` container is limited to using 50% of a single CPU core.

Resource constraints help prevent situations where one container consumes too many resources and impacts the performance of other containers or the Docker host.


\
**2. How can you run only a specific service from `docker-compose.yml` without running the others?**  

To run only a specific service from `docker-compose.yml` without running the rest, you can use the command:
```commandline
docker-compose up service_name
```
Replace `<service_name>` with the name of the service you want to run.
