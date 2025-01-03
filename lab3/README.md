## Task 3. Kubernetes

Install Kubernetes on the local machine (example given for Minikube on Windows 10/11). Deploy a test service.

**Steps:**
- Install Minikube
- Create objects via CLI
- Connect externally

**Additionally, make some modifications to the example manifests to achieve the following:**
- For Postgres, move `POSTGRES_USER` and `POSTGRES_PASSWORD` from the ConfigMap to a Secret (obviously, a new manifest for the Secret resource is needed)
- For Nextcloud, move its variables (`NEXTCLOUD_UPDATE`, `ALLOW_EMPTY_PASSWORD`, etc.) from the Deployment into a ConfigMap (obviously, a new manifest for the ConfigMap resource is needed)
- For Nextcloud, add Liveness and Readiness probes

### Run
```commandline
cd lab3
```
___
**Creating objects via CLI**
- Start Minikube and create the YAML files (manifests) for the ConfigMap, Service, and Deployment.
  - Modify the manifests as follows:
      - For Postgres, move `POSTGRES_USER` and `POSTGRES_PASSWORD` from the ConfigMap to Secrets (obviously, a new manifest for the Secret resource is needed)
      - For Nextcloud, move its variables (`NEXTCLOUD_UPDATE`, `ALLOW_EMPTY_PASSWORD`, etc.) from the Deployment into a ConfigMap (obviously, a new manifest for the ConfigMap resource is needed)
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
**Is the order of these manifests important? Why?**

Yes, the order of these manifests is important for the following reasons:
- **pg_configmap.yml** and **postgres-secrets.yml** create a ConfigMap and a Secret needed for configuring and running PostgreSQL. These resources must be created before creating the PostgreSQL Deployment, as the Deployment relies on them to configure the container environment.
- **pg_service.yml** creates the Service that provides network access to the PostgreSQL database within Kubernetes. This is important so that Nextcloud can connect to the database by the service name `postgres-service`.
- **pg_deployment.yml** creates the PostgreSQL Deployment, which depends on the existing ConfigMap and Secret to launch successfully.
- **nextcloud_configmap.yml** creates a ConfigMap for Nextcloud, containing the settings needed for initialization and configuration.
- **nextcloud.yml** creates the Nextcloud Deployment, which uses the ConfigMap and Secret for its configuration and also connects to the PostgreSQL database through the `postgres-service`.

**What happens if you scale the number of replicas in postgres-deployment to 0, then back to 1, and then try to access Nextcloud again? Why?**
- When the PostgreSQL pod stops and starts up again, the database data is not saved. If you try to access Nextcloud after restarting the PostgreSQL pod, Nextcloud will not be able to connect to the database because the data has been lost.
- To ensure data persistence between pod restarts, you need to use a PersistentVolume and PersistentVolumeClaim to store the database data outside of the pod.

![image](/lab3/docs/5.png)
![image](/lab3/docs/6.png)

