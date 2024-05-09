# Kubernetes (k8s) Demo

## What is Kubernetes?
* Open source container orchestration tool
* Developed by Google
* Helps manage containerized applications in different deployment environments

### Need for container orchestration tool
* Trend from Monolith to Microservices
* Increased usage of containers
* Demand for a proper way of managing those hundreds of containers

### What features do orchestration tools offer?
* High Availability or no downtime
* Scalability or high performance
* Disaster recovery - backup or restore

## Kubernetes Architecture
* Master Node or Control Plane Node (One)
  * Handful of master processes
  * Much more important
  * Runs important K8s processes
    - API Server -> Entrypoint to K8s cluster (UI, API, CLI communicates through this process)
    - Controller Manager -> Keeps track of what is happening in the cluster
    - Scheduler -> Ensures Pods placement
    - etcd -> Kubernetes backing store

* Worker Node (Multiple)
  - Runs our applications
  - Higher workload
  - Much bigger and more resources

* Virtual Network
- Turns all the nodes inside a cluster into one powerful machine that has sum of all the resources of the individual nodes.

*Note: Each node has kubelet process running on it. Kubelet is a Kubernetes process that makes it possible for the clusters to talk/communicate to each other and actually execute some tasks on those nodes like running application processes.*

## Main Kubernetes Components
### Node
* Virtual or physical machine

### Pod
* Smallest unit in Kubernetes
* Abstraction over container
* Usually 1 Application per Pod
* Each Pod gets its own IP address
* Pods are ephemeral (which means they can die very easily)
* New IP address on re-creation

*Note: We only interact with the Kubernetes layer*

### Service
* Permanent IP address
* Lifecycle of Pod and Service are not connected
* You specify the type of Service on creation
* Internal Service is the default type

### Ingress
* External Service that is used to communicate to internal services

### ConfigMap
* External Configuration of our application
* It is for non-confidential data only

### Secret
* Used to store secret data in base64 encoded format
* The built-in security mechanism is not enabled by default
* Reference Secret in Deployment/Pod

*Note: Create Secret or ConfigMap just once. Reference as often as we need!*

*Caution: Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment.*
#### In order to safely use Secrets, take at least the following steps:
* Enable Encryption at Rest for Secrets.
* Enable or configure RBAC rules that restrict reading data in Secrets (including via indirect means).
* Where appropriate, also use mechanisms such as RBAC to limit which principles are allowed to create new Secrets or replace existing ones.

### Data Storage (Volume)
* Storage on local machine
* Or remote, outside the K8s cluster
* Kubernetes doesn't manage data persistence

### Deployment
* Blueprint for "my-app" Pods (a template for creating pods)
* We create Deployments
* Abstraction of Pods
* For stateLESS Apps

*Note: ConfigMap and Secret must exist before Deployments*

### StatefulSet
* For STATEFUL Apps or Databases
* Deploying StatefulSet is not easy

*Note: DBs are often hosted outside of Kubernetes cluster*

## Kubernetes Configuration
* Each Configuration File has 3 Parts
  - metadata
  - specification
  - status (automatically generated and added by Kubernetes)
* Example:
```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-deployment
    labels: ...
spec:
    replicas: 2
    selector: ...
    template: ...
```
*Note: etcd holds the current status of any K8s component!*

## Format of K8s Configuration File
### YAML Configuration Files
* "human friendly" data serialization standard for all programming languages
* syntax: strict indentation
* store the config file with your code
* or own git repository

## Minikube
* Open source tool
* One node cluster where master and worker processes both run on one node
* Docker pre-installed

## kubectl
* Command line tool for K8s cluster
* Enables interaction with cluster (API Server)
* The most powerful of 3 clients (UI, API, CLI)

## Installation & create Minikube cluster
* Run Minikube either as a container or Virtual Machine on our laptop
* * Minikube has Docker pre-installed to run the containers in the cluster
* Driver means we are hosting Minikube as a container on our local machine
* Minikube has kubectl as dependency so NO separate installation necessary
* Install Docker
* Command to install Minikube using Homebrew "brew install minikube"

## Minikube Commands
### minikube start --driver docker
* Starts Minikube as Docker container

### minikube status
* Displays status of the Minikube

### minikube ip
* Displays externally accessible IP of the minikube

## kubectl Commands
### kubectl get node
* Displays all the nodes in the cluster

### kubectl apply -f <file-name.yaml>
* Manages applications through files defining K8s resources
* Examples:
  - kubectl apply -f mongo-config.yaml
  - kubectl apply -f mongo-secret.yaml
  - kubectl apply -f mongo.yaml
  - kubectl apply -f webapp.yaml

### kubectl get all
* Displays all the pods created in the cluster

### kubectl get pod | configmap | secret | service or svc | ...
* Display different resources like pods, configmaps, secrets, service, and more.
* "-o wide" can be used to get wide or longer output. Eg. kubectl get node -o wide

### kubectl --help
* Displays all the sub commands we can use with kubectl

### kubectl get --help
* Displays how we can use get command with kubectl

### kubectl describe <resourceType> <resourceName>
* Displays detailed information
* Examples:
- kubectl describe service webapp-service
- kubectl describe pod webapp-deployment-dcffd6bcc-gsmlt

### kubectl logs <podName>
* Display logs of container
* "-f" can be used to stream the logs. Eg. kubectl logs webapp-deployment-dcffd6bcc-gsmlt -f
* Example: kubectl logs webapp-deployment-dcffd6bcc-gsmlt

### Note: Demo project is attached with the repository.

* Reference Video: https://www.youtube.com/watch?v=s_o8dwzRlu4