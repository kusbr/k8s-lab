# Kubernetes Basics Workshop

## Synopsis
In this workshop, you will understand the fundamentals of containers and kubernetes by doing few simple handson exercises using Docker and Kubernetes.

## Prerequisites
Following software must be installed and ready prior to the workshop:
### 1. Development system (Windows or Linux)
- On Windows - [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install), [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/), [VS Code](https://code.visualstudio.com/Download), [Kubernetes-CLI (kubectl)](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/), [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli)
- On Linux client with GUI – [Docker](https://docs.docker.com/desktop/install/linux-install/), [VS Code](https://code.visualstudio.com/Download), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/), [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
- Optional – [Windows Terminal](https://learn.microsoft.com/en-us/windows/terminal/install) on Windows
- Optional – VS Code extensions for [Docker](https://code.visualstudio.com/docs/containers/overview), [Kubernetes](https://code.visualstudio.com/docs/azure/kubernetes)
- Optional – [K9s](https://k9scli.io/topics/install/) (use [choco](https://chocolatey.org/install) toolchain to install on Windows)
### 2. Kubernetes cluster 
You require a K8s cluster - use any of the following options per your convenience and resource availability:

- Option1 (Recommended): On Azure – Install an [AKS](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli) cluster with minimal configuration (e.g. single node D4s) – do NOT deploy the application mentioned in the documentation

- Option2: On Windows with WSL2 – [Kubernetes engine provided by Docker Desktop](https://docs.docker.com/desktop/kubernetes/)  or [KIND](https://kind.sigs.k8s.io/docs/user/quick-start/)
		
- Option3: On Linux – [KIND](https://kind.sigs.k8s.io/docs/user/quick-start/)  or [K3s](https://docs.k3s.io/quick-start)


### 3. Container Registry
You will need a Container registry for the container images.
- On Azure – Provision an Azure Container Registry (ACR) resource – [link](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal?tabs=azure-cli)
		
OR
- On Docker hub – Create an [account](https://hub.docker.com/) if you do not already have one
## Container Fundamentals

Reference: [How containers work](https://learn.microsoft.com/en-us/virtualization/windowscontainers/about/#how-containers-work)

#### Sample Docker Solution
![Docker Solution](docker-solution.png)

### 4. Activity1: Containerize a simple web application

In this activity you will modify the Dockerfile and build container images using Docker for the Ping and Pong Services. You will use the same Dockerfile for both the services and add the specifics such as executable and port number through build arguments.

    Note: Source files path - src/docker

#### 4a. Update the Dockerfile

- Use a jdk base image
        
        FROM openjdk:8-jdk-alpine
- copy the executable to be launched within the container 
        
        COPY ${JAR_FILE} app.jar 

- expose a port

        EXPOSE ${PORT}

Reference:  [Dockerfile](https://docs.docker.com/engine/reference/builder/#from), [FROM](https://docs.docker.com/engine/reference/builder/#from), [COPY](https://docs.docker.com/engine/reference/builder/#copy), [ARG](https://docs.docker.com/engine/reference/builder/#arg), [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose), [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)

#### 4b. Build the Container images

     Note: path src/docker

- Ping Service Image

    Use docker command to build the ping service image by passing the ping.jar as the executable and 8090 as port.

        docker build -t ping:local -f ./dockerfile --build-arg JAR_FILE="./apps/ping.jar" --build-arg PORT="8090" . 

- Pong Service Image

    Build the pong service image by passing the pong.jar as the executable and 9090 as port.

        docker build -t pong:local -f ./dockerfile --build-arg JAR_FILE="./apps/pong.jar" --build-arg PORT="9090" . 

- Check if the images are listed

        Linux
        docker images | grep "local"

        Windows
        docker images | findstr "local"

Reference: [Docker build](https://docs.docker.com/engine/reference/commandline/build/), [image tag](https://docs.docker.com/engine/reference/commandline/build/#tag), [--build-arg](https://docs.docker.com/engine/reference/commandline/build/#build-arg), [build context](https://docs.docker.com/build/building/context/)

#### 4c: Run the Ping and Pong Containers

- Run the ping app container

        docker run -it --rm --name ping -p 8090:8090 ping:local

- Access the [pinger web api](http://localhost:8090/ping/1)

        localhost:8090/ping/1


- Debug the issue inside the ping container

        docker exec -it ping env

- Create a docker network

        docker network create test

- Run the dependency container for the cachedb  (note the --name redis and --net=test)

        docker run -itd --net=test --name redis -p6379:6379 redis

- Run ping in background but interactive mode in the same network and pass the cache container name in the env

         docker run -itd --env CACHE_HOST=redis -p8090:8090 --name ping --net=test ping:local


- View the logs of ping container
                
        docker logs -f  ping

- Access the [pinger web api](http://localhost:8090/ping/1)

        localhost:8090/ping/1

- Delete the ping and redis docker containers

        docker rm -f ping
        docker rm -f redis

#### 4d: Optional - Try similar steps for pong container

        Once the pong container is ready, the app can be accessed at localhost:9090/pong/{key}

### 5. Container Registry

#### 5a-i. Activity: Docker Hub

- Login to docker hub

        docker login -u <username>

OR

#### 5a-ii. Activity: Azure Container Registry (ACR)

- Create an ACR resource in Azure if you have not alrady created as part of pre-requisites - [Link](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal?tabs=azure-cli)

- Login to ACR using Az CLI

        az acr login -n <name of acr>

#### 5b. Tag and Push Images to Container Registry

- Tag

        docker tag ping:local <cr full name>/ping:v1
        docker tag pong:local <cr full name>/pong:v1

- Push to CR

        docker push <cr full name>/ping:v1
        docker push <cr full name>/pong:v1

#### 5c. Pull / Run a CR image

        docker pull <cr full name>/ping:v1

        e.g.    docker pull docker.io/kumsub/ping:v1
                docker pull acrlabdev.azurecr.io/ping:v1

        docker run --rm pingtest <cr full name>/ping:v1

## Kubernetes Fundamentals

### Sample Kubernetes Solution 
![K8s Solution](k8s-samplesolution.png)

### 6. Activity: Kubernetes Cluster Context

#### 6a. If you are using Kubernetes engine provided by Docker-Desktop on Windows WSL2, then set the context to the local cluster "docker-desktop"

        kubectl config use-context docker-desktop

#### 6b. If you are using Azure Kubernetes Service (AKS), use Az CLI to get credentials and set the context to the AKS cluster

        az aks get-credentials -n <aks-cluster-name> -g <rg-name>

#### 6c. Verify that the context is set to the correct cluster

        kubectl config get-contexts

### 7. Activity: Create a kubernetes namespace

Reference: [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

        Source Path: src/k8s/1.namespace/
        File: ns.yaml

Use kubernetes CLI kubectl to create a new namespace "pingpong"

        kubectl create -f ns.yaml

        kubectl get ns

### 8. Activity: Deploy the Pod for Ping App

Reference: [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

You will deploy the ping app using a "pod" manifest

        Source path: src/k8s/2.ping/
        File: pod.yaml

- Modify the pod.yaml to set the docker image name for the container and the namespace

- Schedule the ping pod. 
Verify if the pod is created and see the container logs

        kubectl create -f pod.yaml -n pingpong

        kubectl get pods -n pingpong

        kubectl -n pingpong describe pod pingapi

- Try accessing the ping api as before using pod ip

        http://{pod ip}:8090/ping/1

### 9. Activity: Image Pull Credentials
If your container images are in a private regisry such as ACR, Kubernetes will need the permission to pull the images during pod scheduling. There are multiple scenario based options:

#### Option1: AKS Attach - Attach AKS Cluster to ACR
- [Existing AKS Cluster](https://learn.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?toc=%2Fazure%2Fcontainer-registry%2Ftoc.json&bc=%2Fazure%2Fcontainer-registry%2Fbreadcrumb%2Ftoc.json&tabs=azure-cli#configure-acr-integration-for-an-existing-aks-cluster)

- [New AKS Cluster](https://learn.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?toc=%2Fazure%2Fcontainer-registry%2Ftoc.json&bc=%2Fazure%2Fcontainer-registry%2Fbreadcrumb%2Ftoc.json&tabs=azure-cli#tabpanel_2_azure-cli)

#### Option2: CR Credentials
Store the CR credentials in a K8s Secret and use the secret as ImagePullSecret in the manifest

- Create K8s [Secret](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)

- Update the pod.yaml / deployment.yaml to use the secret in the spec

        imagePullSecrets:
        - name: {K8s secret name that has the CR creds}

#### Option3: Azure MSI based Workload Identity
We will not try this option as part of this lab. You can do this as part of your post lab learning
- [AKS MSI](https://learn.microsoft.com/en-us/azure/aks/use-managed-identity)

- [AcrPull Role](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-roles?tabs=azure-cli#pull-image) - Assign to MSI

### Activity 10. Deploy a K8s service for the Ping Api

Reference: [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

        Source path: src/k8s/2.ping
        File: service.yaml

- Modify the service.yaml to set the service type to "LoadBalancer", and submit the manifest to the K8s cluster

        kubectl -n pingpong create -f service.yaml

        kubectl -n pingpong get services

        Note: Wait for the external IP (in case of AKS) to be available

- Describe the service to check if it is bound to the container endpoint

        kubectl -n pingpong describe service pingservice

- If the EndPoints value is None, no container is bound to the service and no routing will happen for the service IP  

- Update the pingapi pod.yaml metadata to set the tier label  to "frontend"

        kubectl -n pingpong apply -f pod.yaml

- Access the ping api using the service's public endpoint

        http://<external-ip>:8090/ping/1

- Let's get inside the ping container and access the api endpoint locally

        kubectl exec -ti pingapi -n pingpong -- /bin/sh

        wget http://localhost:8090/ping/1
        
- Check the pod logs

        kubectl logs -f pingapi -n pingpong

### 11. Activity: Deploy the Cache DB

Reference: [Kubernetes Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

        Source Path: src/k8s/3.cache
        Files: deployment.yaml, service.yaml

- Submit a redis deployment and a internal service to the cluster

        kubectl -n pingpong create -f ./

        kubectl -n pingpong get pods
        
        kubectl -n pingpong get svc redis

        Note: No external IP

- Scale the cache replicasets to 3 replicas

        kubectl -n pingpong scale deployment redis --replicas=3

        kubectl -n pingpong get rs 

        kubectl -n pingpong get pods

- Check the ping service http://<external-ip>:8090/ping/1

### 12. Optional Activity:  Cross Namespace Service Access

Deploy redis to a different namespace and make the ping service work with that cache

- Delete redis deployment and service in pingpong ns
- Create redis deployment in another ns, say "cache"

        Hint: Access pattern: {servicename}.{ns}.svc:port

### 13. Activity: Deploy the Pong Service

        Source Path: src/k8s/4.pong

- Submit the deployment and service yamls
- Check if the pong service is accessible

        http://{pong-service-public-ip}:9090/pong/{key}

### 14. Service Accounts and Volumes