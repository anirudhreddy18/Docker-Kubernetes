# DOCKER AND KUBERNETES

## DOCKER
* docker run -it redis (run redis commands in a terminal)
* Image --> Single file with all dependecies and config required to run a program
* Container -->  Runtime instance of an image

### Docker Ecosystem
* Docker client (running commands or docker cli)
* Docker server (responsible for creating images, running containers etc)
* Docker machine
* Docker Images
* Docker compose
* Docker Hub

*** Docker client does some processing on commands enetered and docker server does the heavy lifting
*** Docker server looks for image cache first if it dosent find then downloads from priavte repo/docker hub.
when running docker on mac/windows it actually installs a Linux VM and then runs processes in the VM.

docker logs container-id (Get logs of a container)
docker stop container-id || docker kill container-id
docker rm container-id
docker ps || docker ps --all

Redis 
docker run redis (Runs the redis server on port 6379 in a container)
To connect to a container then use docker exec -it container-id redis-cli 

Terminal in Container
docker exec -it container-id sh|bash (Run command prompt in a container)
also can use docker run -it image sh
containers always run in isolation
docker container prune (deletes all containers)

DockerFile
base image  FROM
Run some commands  RUN
commnad to run on container startup  CMD
docker build -t avemula/nginx:latest .

SAMPLE NODE JS APP
docker run -it avemula/sample-node sh
docker exec -it container-id sh
ls will show all the files
exit to get out of container or ctrl+c or ctrl+d
copy package.json and run npm install and then copy all remaining  this will allow cache for the dependecies and then if u change index.js file build would be fast

DOCKER COMPOSE
docker-compose up -d
docker-compose down
restart: always --> always restarts the containers when container start fails
docker-compose ps (similar to docker ps)

-f to specify different docker file (-f Dockerfile.dev)
React APP
Start react app in local docker container
docker run -p 3000:3000 -v $(pwd):/app image-id -v /app/node_modules (if we want to use container node modules)
docker exec -it container-id npm run test(can specify different commands to running container)

## Kubernetes
- what is kubernetes?  system for running many different containers over multiple VM's.
- why use kubernetes? when you need to run many different containers with different images.

### Uses:
- Can scale up individual services instead of whole app (client, nginx, server) we could just increase server instances.
- Running many different containers in multiple different virtual machines.

- Development --> mini cube (command line tool)
- Production --> Amazon elastic container service, google cloud Kubernetes engine

- (installed kubectl, virtualBox, miniKube for local Development).
- **minikube to used to install kubernetes cluster, and kubectl is used to interact with the cluster. (Local Development)**
- **minikube start (installed a vm on local conmputer) which is called Node.**

### Checking if miniKube & Kubectl Started
- kubectl cluster-info (checking status)
- miniKube status
- minikube ip
- minikube dashboard
- minikube stop

- config File (used to  create objects) ==> StatefulSet, ReplicaContainer, Pod, Service (running a container, monitoring a container, setting up networking)
- **Pod means running 1 or more closely related containers.** 
- **Service means setup Networking in Kubernetes Cluster.**
- **Deployment maintains set of identical pods, ensuring they have correct config and right number exists.**
- **Smallest deployable unit in kubernetes is a Pod(a pod has to have 1 or more containers).**

### Object Types
- Pod (used for Development but we always deploy pods using deployment file)
- Deployment (used in Production since we can update replicas, image, container name, container Port)

### Service (Setup Networking) Types 
ClusterIP (expose access of pods for other objects inside the cluster like API connects to redis/postgres, not exposed to outside traffic, only exposed in cluster)
NodePort (Expose container/pod to outside world only for development) 
LoadBalancer (Legacy because we want UI & API via nginx both exposed to outside world & load balacer can only do it for 1 pod)
Ingress (accepts incoming traffic & routing rules to get traffic to services ex: nignx ingress)

### YAML File
- Selector component in node port file is used to identify how to route traffic to a pod. (labels is how its defined in a pod file) nodePort would be in b/w (30000 - 32767)

## All Kubectl Commands
- kubectl apply -f filename (we will always interact with master node & feed commands to master)
- kubectl delete -f filename (will remove the pod)
- kubectl delete deployment name
- kubectl delete service name

### Get Status of deployed Pods / services/deployments
- kubectl get pods -o wide
- kubectl get services
- kubectl get deployments
- kubectl get pv (to get Persisternt Volume Claim)
- kubectl describe pod/object name-of-object
- kubectl logs pod-name/service-name (similar to docker ps)
- kubectl get secrets

- All Kubectl commands are feeded to master node. we dont really touch the node/VM master takes care of it.
- Each node/VM has a docker preinstalled , downloads the image from docker hub & runs the image in a container.
- We will be following declarative approach all the time.

- Updating a config file (have the same name & type always & then update remaining things). Name is the unique ID, if its a new name then master will create a new pod.

- to need details for an object command is kubectl describe pod name-of-object 

### Deployment pod Template
- containers: 1  
- name: client (container name)
- port: 3000   (port number)
- image: multi-client (docker image)

- Labels are the only way to identify which pod we are referring to. (selector in service object and matchLables in deployment object matches that)
- PODS IP ADDRESS keep changing so we therefore we use service objects to connect to the pod

### How to recreate our pods with latest version of multi-client image pushed to docker hub
1. push updated docker image to docker hub with new version & update kubernetes config file with new version.
2. Imperative command push updted image to docker hub with tag & apply this command
3. kubectl set image deployments/client-deployment client=anirudhreddy18/multi-client

*** eval $(minikube docker-env) ---- connects to docker server in kubernetes node (runs only in current terminal) why? still use docker cli commands for debugging

### Volumes (Different than docker volumes where container data is mapped to file system on host/VM/laptop )
- Volume -- An object that allows a container to store data at a pod level. (can't use this because if a pod crashes data is lost)
- Persistent Volume -- if a pod crashes, volume is still there & new pod can connect to the same data
- Persistent Voulme Claim -- like an advertisement where u can ask for different hard drive options, kubernetes already has statically provisioned volume if its not readily available, then 
  provision a dynamically provisioned voulme. Master will create PVC in the cluster/node.

*** API wants to connect to redis/postgres HOST/URL would be name of clusterip service of postgres/redis     

### Create Secrets
- kubectl create secret generic pgpassword --from-literal PGPASSWORD=12345asdf

### INGRESS
We are using kubernetes/nginx-ingress to configure to outside traffic.
Deployment is a type of controller since we are updating state, like initially 0 pods running & applying config file to master changes to 3 pods running.

## Deployment to Cloud GCP 
- CI RUnner Steps
- install cli google cloud
-  auth info google cloud
- Run Tests 
- docker cli login
- build new images with tags                          docker build -t anirudhreddy18/image:latest -t anirudhreddy18/image:1.2.3 .
- push to docker hub                                  docker push anirudhreddy18/image:latest     docker push anirudhreddy18/image:1.2.3 
- apply config files k8s folder                       kubectl apply -f k8s
- Imperatively set images on each deployment.         kubectl set image deployments/deployment-name containerName=anirudhreddy18/image:1.2.3 
- 1 advantage of using GCP is we can connect to cluster via terminal and run same kubectl commands.

### Kubernetes engine
- Clusters --:> cluster
- Workloads -> deployments, Pods
- Services: ClusterIP, Load Babalancer, Ingress
- Configuration: Secrets

### Helm
- Local mac install by using Homebrew
- Helm 3rd party software  which has Helm CLient connects to tiller Server
- RBAC (Role based Access Control) UserAccounts , Service Accounts 
- Roles -- ClusterRoleBinding, Role Binding
- To install nginx Ingress we are using Helm.

## Kubernetes Concepts
  - control plane
  - worker Nodes

### Control Plane 
 -  responsible for maintaining state of cluster. 
 - Pods are created & managed by control plane.
   
### Master:
 - API Server -> primary interface b/w control plane & rest of the cluster, rest api.
 - etcd -> is a key value store, store the persistent state of the cluster
 - scheduler -> scheduling pods on worker nodes
 - controller manager -> run controllers to maintain state like replication, rollout etc.

### Worker
  - container runtime -> docker
  - kube-proxy: routing traffic b/w pods & load balancing
  - kubelet: deamon running, maintains desired state from pods, receive instructions from control plane.
 - Managed Service providers GKE, EKS manage control planes

UI/cli ————sends rest api request ——> Control plane ——> API Server —————> Worker Nodes

### Features:
  - Networking
  - service Discovery
  - Self Healing
  - Scaling
  - Load balancing
  - Automated Rollbacks.

### Patterns
 - Ambassador - proxy -> provides https support.
 - Adapter -> volume sharing , one container mounts at particular location & other takes that as input & adapts.
 - Init -> fetch secrets from vault
 - Sidecar => monitoring , authentication, logging etc
