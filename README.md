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
### Master Node or Control Plane Node (Usually One)
  * Manages the processes (container runtime, kubelet, and kube proxy)
  * Handful of master processes
  * Much more important
  * Runs important K8s processes
    - API Server
      * Entrypoint to K8s cluster (UI, API, CLI communicates through this process)
      * Cluster gateway and is load balanced
      * acts as a gatekeeper for authentication
    - Scheduler 
      * Ensures Pods placement
      * Just decides on which Node new Pod should be scheduled
    - Controller Manager
      * Keeps track of what is happening in the cluster
      * Detects cluster state changes
    - etcd
      * Kubernetes backing store
      * etcd is the cluster brain
      * Cluster changes get stored in the key value store
      * Application data is NOT stored in etcd

### Worker Node or Data Plane Node (Usually Multiple)
  * Runs our applications
  * Higher workload
  * Much bigger and more resources
  * Virtual Network
  * Turns all the nodes inside a cluster into one powerful machine that has sum of all the resources of the individual nodes.

*Note: Each node has kubelet process running on it. Kubelet is a Kubernetes process that makes it possible for the clusters to talk/communicate to each other and actually execute some tasks on those nodes like running application processes.*

## Layers of Abstraction
1. Deployment manages a ...
2. ReplicaSet manages a ...
3. Pod is an abstraction of ...
4. Container

*Note: Everything below Deployment is handled by Kubernetes*

## Main Kubernetes Components
### Node
* Virtual or physical machine
* Each node has multiple pods on it
* 3 processes (container runtime, kubelet, and Kube proxy) must be installed on every node
* Worker node do the actual work
* For high availability it is better to configure at least 3 master nodes
* As needed master and worker nodes can be added
* Kubelet
  - interacts with both - the container and node
  - starts the pod with a container inside
* Kube Proxy
  - forwards the requests

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
* Abstraction over Pods
* For stateLESS Apps

*Note: ConfigMap and Secret must exist before Deployments*

### StatefulSet
* For STATEFUL Apps or Databases
* Deploying StatefulSet is not easy

*Note: DBs are often hosted outside of Kubernetes cluster*

## Kubernetes Namespace
* Kubernetes cluster organize resources in namespaces
* Virtual cluster inside a kubernetes cluster
* `kubectl get namespace` returns the list of namespaces
* kubernetes-dashboard namespace
  - only with minikube
* kube-system namespace
  - not meant for our use
  - Do NOT create or modify in kube-system
  - System processes uses it
  - Master and kubectl processes uses it
* kube-public
  - publicly accessible data
  - A configmap, which contains cluster information
  - `kubectl cluster-info` gives the output with information
* kube-node-lease namespace
  - Holds information about heartbeats of nodes
  - Each node has associated lease object in namespace
  - Determines the availability of a node
* default namespace
  - Resources we create are located here if we haven't created any new namespace

### Why to use Namespaces?
* Main reasons for using namespaces:
  - Structure our components
    * Difficult overview of everything inside default namespace
  - Avoid conflicts between teams
    * Conflicts: Many teams, same application like one team can override other teams deployment
  - Share services between different environments
    * Resources Sharing: Staging and Development like breaking down into Staging, Development namespaces can share common Nginx-Ingress-Controller and Elastic Stack namespaces
    * Resources Sharing: Blue/Green Deployment like Production-Blue and Production-Green namespaces can share common Nginx-Ingress-Controller and Elastic-Stack namespaces
  - Access and Resource Limits on Namespaces Level
    * Access and Resource Limits on Namespaces like one team's namespace cannot access resources from another team's namespace i.e. each team has own, isolated environment. As well as, the resources (CPU, RAM, Storage) can be limited per namespace for different teams.
* Example of Resources grouped in Namespaces:
  - Database namespace
  - Monitoring namespace
  - Elastic-Stack namespace
  - Nginx-Ingress namespace

*Note: Officially we should not use namespaces for smaller projects i.e. with less than 10 users*

### Characteristics of Namespaces?
* We can't access most of the resources from another Namespace
  - ConfigMap in ProjectA namespace cannot reference DatabaseCerts from ProjectB namespace
  - Each NS must define own ConfigMap
  - Same applies to Secret
  - However, service resources can be shared between different namespaces
  - Example: in the following example `mysql-service` is service name and `database` is namespace for `db_url`.
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mysql-configmap
  data:
    db_url: mysql-service.database
  ```
  - Components, which can't be created within a Namespace
    * live globally in a cluster
    * we can't isolate them
    * Examples of such components are volume or persistent volume and node
    * These components can accessible throughout the whole cluster since they are not bound to specific namespace
    * Command to list all the resources that are NOT bound to namespaces
    ```shell
    kubectl api-resources --namespaced=false
    ```
    * Command to list all the resources that are bound to namespaces
    ```shell
    kubectl api-resources --namespaced=true
    ```

### Command to create namespace
* `kubectl create namespace [namespace_name]`
* Example:
```shell
kubectl create namespace my-namespace
```

### Create component in a Namespace
* By default, components are created in a default NS
* We can specify the namespace by using `--namespace` parameter in the `kubectl apply` command or specifying a parameter `namespace` under `metadata` in the YAML file.
* Example:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: mysql-configmap
namespace: my-namespace
data:
db_url: mysql-service.database
```
```shell
kubectl apply -f mysql-configmap.yaml --namespace=my-namespace
```

*Note: It is a better practice to specify namespace in the YAML configuration file so that it is better documented and more convenient*

### Change active namespace
* We can change the active namespace with kubens!
* Command to install kubens tool
```shell
brew install kubectx
```
* Command to list all the namespaces
```shell
kubens
```
* Command to change active namespace `kubens [namespace_name]`
```shell
kubens my-namespace
```

## Kubernetes Ingress
### External Service VS Ingress
* External Service assigns external IP address to service
* Ingress has its own kind called `Ingress` and has Routing rules that binds the internal service where to forward the requests to

### Ingress Configuration
* Host
  - valid domain address
  - map domain name to Node's IP address, which is the entrypoint
* Example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: dashboard.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-internal-service
                port:
                  number: 8080
```

### Multiple paths for same host using Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /analytics
            pathType: Prefix
            backend:
            service:
              name: analytics-service
              port:
                number: 3000
          - path: /shopping
            pathType: Prefix
            backend:
            service:
              name: shopping-service
              port:
                number: 8000
```

### Multiple sub-domains or domains using Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
    - host: analytics.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
            service:
              name: analytics-service
              port:
                number: 3000
    - host: shopping.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
            service:
              name: shopping-service
              port:
                number: 8000
```

### How to configure Ingress in our K8s cluster?
* We need an implementation for Ingress called Ingress Controller
* Ingress Controller
  - evaluates and processes Ingress rules
  - evaluates all the rules
  - manages redirections
  - entrypoint to cluster
  - many third-party implementations, there is one from Kubernetes called K8s Nginx Ingress Controller
* Environment on which our cluster runs
  - Cloud service provider: AWS, Google Cloud, Linode
    * Out-of-the-box K8s solutions
    * Own virtualized Load Balancer
    * External request comes to Cloud Load Balancer which forwards it to Ingress Controller
    * But we can configure it in different ways
    * Advantage: We don't have to implement load balancer by ourselves
  - Bare Metal:
    * We need to configure some kind of entrypoint
    * Either inside of cluster or outside as separate server, we have to provide entrypoint like External Proxy Server (Software or hardware solution)
    * separate server
    * public IP address and open ports
    * entrypoint to cluster

*Note: No server in K8s cluster is accessible from outside.*

### Configuring TLS Certificate - https//
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
    - hosts:
        - myapp.com
        secretName: myapp-secret-tls
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
            service:
              name: myapp-internal-service
              port:
                number: 8080
---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

#### Configure Ingress in Minikube
* Install Ingress Controller in Minikube
  ```shell
  minikube addons enable ingress
  ```
  - Automatically starts the K8s Nginx implementation of Ingress Controller
  - Ingress Controller configured with just 1 command :)
* View the Ingress Controller Pod
  ```shell
  kubectl get pod -n ingress-nginx
  ```
* Enable Minikube Dashboard
  ```shell
  minikube dashboard  
  ```
* View the resources from kubernetes-dashboard namespace
```shell
kubectl get all -n kubernetes-dashboard
```
* YAML File To Expose Kubernetes Dashboard:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
spec:
  rules:
    - host: dashboard.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-dashboard
                port:
                  number: 80
```
* Commands to apply dashboard-ingress and view its external IP:
```shell
kubectl apply -f dashboard-ingress.yaml
kubectl get ingress -n kubernetes-dashboard
```
* Copy the external IP from the above command to view the IP of the ingress and add the following line by replacing <EXTERNAL_IP> to the `/etc/hosts` file
```
<EXTERNAL_IP>   dashboard.com
```
* Configure Default Backend which is used to display some meaningful message to the user if the page is not found like trying to access dashboard.com/abc
```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
spec:
  selector:
    app: default-response-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

## Helm
* It is a package manager for Kubernetes
* To package YAML Files and distribute them in public and private repositories
* First Feature: Sharing Helm Charts
* Second Feature: Templating Engine
* Third Feature: Deployment of Same Applications across different Environments (Development, Staging, Production)
* When `helm install <chartname>` ins executed:
  - Template files will be filled with the values from values.yaml

### Helm Charts
* Bundle of YAML Files
* Create our own Helm Charts with Helm
* Push them to Helm Repository (Private or Public)
* Download and use existing ones
* Commonly used deployments like Database Apps (ElasticSearch, MongoDB, MySQL), Monitoring Apps (Prometheus) already have charts available in Helm repository
* We can reuse that charts
* Sharing Helm Charts made Helm popular
* Structure of Helm Chart:
  ```
  mychart/
    Chart.yaml
    values.yaml
    charts/
    templates/
    ...
  ```
  - Top level mychart folder is a name of chart
  - Chart.yaml contains meta info about chart like name, dependencies, version
  - values.yaml holds the values for the template files, default values can be overwritten
  - charts folder holds chart dependencies
  - templates folder holds the actual template files
  - Optionally other files like Readme or license file

### Templating Engine
* Define a common blueprint
* Dynamic values are replaced by placeholders
* {{ .Values... }} comes from another external values YAML file
* {{ .Values... }} is an object, which is created based on the values defined in the values YAML file or with `--set` flag
* Instead of having demo YAML files for each microservice, we can replace those values dynamically
* Practical for CI/CD, in our build we can replace the values on the fly 
* Example Template YAML Config:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Values.name }}
spec:
  containers:
  - name: {{ .Values.container.name }}
    image: {{ .Values.container.image }}
    port: {{ .Values.container.port }}
```
* Example values YAML file:
```yaml
name: my-app
container:
  name: my-app-container
  image: my-app-image
  port: 9001
```

### Values injection into template files
1. Using values YAML file (default)
2. Passing the values YAML file in the helm command (override values)
```shell
helm install --values=my-values.yaml <chartname>
```
3. Using `--set` flag OR on Command Line:
```shell
helm install --set version=2.0.0
```

*Note: It is more organized and better manageable to have files (i.e. my-values.yaml) to store all those values instead of just providing it from the command line.*

### Release Management
* Helm Version 2 comes in two parts:
  1. CLIENT (helm CLI)
  2. SERVER (Tiller)
* Whenever we deploy helm chart using `helm install <chartname>`
  - helm CLI will send requests to Tiller that actually has to run on Kubernetes Cluster
  - Tiller then executes those requests and create components from the YAML files inside the Kubernetes Cluster
* Whenever we create or change deployment, Tiller will store a copy of each configuration Client sends for future reference
* Keeping track of all chart executions
  ```shell
  helm install <chartname>
  heml upgrade <chartname>
  ```
  - Changes are applied to existing deployment instead of create a new one.
  - We can apply rollback in case upgrade goes wrong by using the following command
  ```shell
  helm rollback <chartname>
  ```

#### Downsides of Tiller
* Tiller has too much power inside K8s cluster
* Security Issue
* In Helm Version 3 Tiller got removed solving the security concern

## Kubernetes Volumes
* Persistent Volume (pv)
* Persistent Volume Claim (pvc)
* Storage Class (sc)

### The need for Volumes
* No data persistence out of the box
* We need to configure data persistence
* Storage Requirements:
  1. Storage that doesn't depend on the pod lifecycle
  2. Store must be available on all nodes
  3. Storage needs to survive even if cluster crashes
* Persistence Volume
  - a cluster resource (like RAM or CPU)
  - created via YAML file
    * kind: PersistentVolume
    * spec: specifies detailed spec for the pv
  - Needs actual physical storage, like local disk, nfs server, cloud-storage
  - We need to create and manage pv by ourselves
  - External plugin to our cluster
  - We can use that physical storages in the spec section
* Depending on storage type, spec attributes differ
* YAML Example:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-name
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.0
  nfs:
    path: /dir/path/on/nfs/server
    server: nfs-server-ip-address
```
* YAML Example (Google Cloud as a storage backend):
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
  labels:
    failure-domain.beta.kubernetes.io/zone: us-central1-a__us-central1-b
spec:
  capacity:
    storage: 400Gi
  accessModes:
  - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
```
* YAML Example (Local storage):
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  modeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

*Note: Persistent Volumes are NOT namespaced. PV lives outside the namespace and are accessible to whole cluster*

* Complete list of storage backends supported by K8s: https://kubernetes.io/docs/concepts/storage/volumes/

### K8s Administrator and K8s User
* PV are resources that need to be there BEFORE the Pod that depends on it is created
* K8s Admin sets up and maintains the cluster and make sure of enough resources
* Storage resource (nfs-storage or cloud-storage) is provisioned by Admin based on the information from Development Team
* K8s User deploys applications in cluster either directly or using CI pipeline

### Persistent Volume Claim component
* Application has to claim the Persistent Volume
* Example YAML:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-name
spec:
  storageClassName: manual
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
* We have to use that PVC in Pods configuration
* Example YAML:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: pvc-name
```

### Levels of Volume Abstractions
* Pod requests the volume through the PV claim
* Claim tries to find a volume in cluster
* Volume has the actual storage backend

### Why so many abstractions
  - Admin provisions storage resource
  - User creates claim to PV

*Note: Claims must be in the same namespace*

### Local VS Remote Volume Types
* Each volume type has its own use case
* Local volume types violate (ii) and (iii) requirement for data persistence
  - being tied to 1 specific node
  - surviving cluster crashes
  - Data should be safely stored
  - Don't want to set up the actual storages ourselves
  - Makes it easier for developers

*Note: For DB persistence always use remote storage*

### ConfigMap and Secret
* Both of them are local volumes
* Not created via PV and PVC
* Managed by Kubernetes
* Files like configuration file or certificate file need to be mounted inside our application. For this:
  - Create ConfigMap and/or Secret component
  - Mount that into our pod/container

### Different volume types in a Pod
```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: elastic
spec:
  selector:
    matchLabels:
      app: elastic
  template:
    metadata:
      labels:
        app: elastic
    spec:
      containers:
      - imagge: elastic:latest
        name: elastic-container
        ports:
        - containerPort: 9200
        volumeMounts:
          - name: es-persistent-storage
            mountPath: /var/lib/data
          - name: es-secret-dir
            mountPath: /var/lib/secret
          - name: es-config-dir
            mountPath: /var/lib/config
        volumes:
        - name: es-persistent-storage
          persistentVolumeClaim:
            claimName: es-pv-claim
        - name: es-secret-dir
          secret:
            secretName: es-secret
        - name: es-config-dir
          configMap:
            name: es-config-map
```

### Storage Class
* Another abstraction level
  - abstracts underlying storage provider
  - parameters for that storage
* Storage Class provisions Persistent Volumes dynamically when PersistentVolumeClaim claims it.
* StorageBackend is defined in the Storage Class component via `provisioner` attribute
  - each storage backend has own provisioner
  - internal provisioner - `kubernetes.io`
  - external provisioner
* Configure `parameters` for storage we want to request for Persistent Volume

### Storage Class usage
* Requested by PersistentVolumeClaim
* PersistentVolumeClaim Config YAML Example
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - RreadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassname: storage-class-name
```
* StorageClass Config YAML Example
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-class-name
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

### Levels of Volume Abstractions (Using StorageClass)
* Pod claims storage via PersistentVolumeClaim
* PersistentVolumeClaim requests storage from StorageClass
* StorageClass crates PersistentVolume that meets the needs of the Claim

## Kubernetes StatefulSet
* StatefulSet for stateful applications
* Stateful applications
  - Keep record of state
  - Examples: databases, app that stores data, MongoDB (1. update data based on previous state / query data, 2. depends on most up-to-date data/state)
* Stateless applications
  - Don't keep record of state
  - each request is completely new
  - Examples: NodeJS (1. doesn't depend on previous data, 2. pass through for data query/update)

### Deployment of stateful and stateless applications
* stateless apps are deployed using Deployment component
* stateful apps are deployed using StatefulSet component
* Both manage Pods based on container specification
* Configure storage the same way

### Pod Identity
* sticky identity for each pod
* created from same specification, but not interchangeable
* every pod has it own identifier
* persistent identifier across any re-scheduling
* StatefulSet with 3 replica example:
  - MASTER (mysql-0)
  - SLAVE (mysql-1)
  - SLAVE (mysql-2)
* Scaling Up of StatefulSet
  - Next Pod is only created, if previous is up and running
* Scaling Down of StatefulSet
  - Deletion in reverse order, starting from the last one
* These all mechanisms are to protect data and state

### Pod Endpoints
* ${pod name}.${governing service domain}
* Example:
  - mysql-0.svc2
  - mysql-1.svc2
  - mysql-2.svc2

### Characteristics
1. Predictable pod name like mysql-0
2. Fixed individual DNS name like mysql-0.svc2
* When Pod restarts:
  - IP address changes
  - name and endpoint stays same
* Sticky identity makes sure that each replica pod retain its state and role
* Replicating stateful apps is complex but Kubernetes helps us
* We still need to do a lot:
  - Configuring the cloning and data synchronization
  - Make remote storage available
  - Managing and back-up

*Note: Stateful applications are NOT perfect for containerized environments*

## Kubernetes Services (svc)
* Each Pod has its own IP address but, they are ephemeral (are destroyed frequently)
* Is a stable IP address to access the Pod
* Provides load-balancing
* Are a good abstraction for loose coupling within & outside cluster

### ClusterIP Services
* It is a default type so no need to specify type
* Is only accessible within cluster
* Service Communication YAML Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  ...
spec:
  selector:
    app: mongodb
  ports:
    - name: mongodb
      protocol: TCP
      port: 27017
      targetPort: 27017
```
* Multi port Service YAML Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  ...
spec:
  selector:
    app: mongodb
  ports:
    - name: mongodb
      protocol: TCP
      port: 27017
      targetPort: 27017
    - name: mongodb-exporter
      protocol: TCP
      port: 9216
      targetPort: 9216
```

### Headless Services
* Client wants to communicate with 1 specific Pod directly or Pods want to talk directly to specific Pod
* So, not randomly selected since it needs to go to specific Pod
* Use Case: Stateful applications, like databases since Pod replicas are not identical
* In these cases, client needs to figure out IP addresses of each Pod
  - Option 1 - API call to K8s API Server
    * makes app too tied to K8s API
    * inefficient since we need to get all Pods and their IP address
  - Option 2 - DNS Lookup
    * DNS Lookup for Service - returns single IP address (ClusterIP)
    * Set ClusterIP to "None" - returns Pod IP address instead, no cluster IP address will be assigned
* Example YAML File
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-db-service-headless
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

### NodePort Services
* It is an extension of ClusterIP Service
* External traffic has access to fixed port on each Worker Node
* `nodePort` value has predefined range between `30000 - 32767`
* ClusterIP Service is automatically created
* This type of service exposure is not very efficient and secure
* Because external clients basically have access to worker nodes directly
* Example YAML File
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ms-service-nodeport
spec:
  type: NodePort
  selector:
    app: microservice-one
  ports:
    - protocol: TCP
      port: 3200
      targetPort: 3000
      nodePort: 30008
```

*Note: In the real Kubernetes setup example, we would probably not use NodePort Service for external connection.<br/>We would mainly use NodePort Service to test some service quickly but not for production use cases.<br/>Configure Ingress or LoadBalancer for production environments.*

### LoadBalancer Services
* It is a better alternative to NodePort Service
* It is an extension of NodePort Service
* Becomes accessible externally through cloud providers LoadBalancer
* NodePort and ClusterIP Service are created automatically
* Example YAML File
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ms-service-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: microservice-one
    ports:
      - protocol: TCP
        port: 3200
        targetPort: 3000
        nodePort: 30010
```

### Service type attributes
1. ClusterIP (Default, type not needed, internal service)
2. NodePort
3. LoadBalancer

## Kubernetes Configuration
* Each Configuration File has 3 Parts
  - metadata
  - specification
  - status (automatically generated and added by Kubernetes)
* Example:
```yaml
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

## Add new Master/Node server
* get new bare server
* install all the master/worker node processes
* add it to the cluster

## Production Cluster Setup
* Multiple Master and Worker nodes
* Separate virtual or physical machines

## Minikube (For Local Setup)
* Open source tool with Docker pre-installed
* One node cluster where master and worker processes both run on one node (1 Node K8s cluster)
* Crates Virtual Box on our laptop
* Node runs in that Virtual Box
* It is used for testing purposes

## KUBECTL
* Command line tool for K8s cluster
* Enables interaction with cluster (API Server)
* The most powerful of 3 clients (UI, API, CLI)
* Enable pods to run on node (create pods, destroy pods, create services, etc.)

## Installation & create Minikube cluster
* Run Minikube either as a container or Virtual Machine on our laptop
* Minikube has Docker pre-installed to run the containers in the cluster
* Driver means we are hosting Minikube as a container on our local machine
* Minikube has kubectl as dependency so NO separate installation necessary
* Install Docker
* Command to install Minikube using Homebrew
```
brew install minikube
```

## Minikube Commands
### minikube start --driver docker
* Starts Minikube as Docker container

### minikube status
* Displays status of the Minikube

### minikube ip
* Displays externally accessible IP of the minikube

### minikube stop
* Stops the minikube

### minikube service [service_name]
* Assigns the external service a public IP
* Example:
```shell
minikube service mongo-express-service
```

### minikube addons enable ingress
* Enables the ingress addon on Minikube

### minikube dashboard
* Enabled Minikube Dashboard (GUI Interface) if not already enabled and loads it the browser

## kubectl Commands
### kubectl create deployment NAME --image=image [--dry-run] [options]
* Creates deployment
* Blueprint for creating pods
* Most basic configuration for deployment (name and image to use)
* Examples:
```shell
kubectl create deployment nginx-depl --image=nginx
kubectl create deployment mongo-depl --image=mongo
```

### kubectl get replicaset
* Replicaset is managing the replicas of a Pod
* In practice, we never have to create/delete/update replicaset, we would be working with Deployment directly

### kubectl edit deployment [name]
* Allow editing auto-generated configuration file of Deployment with default values 
* Example:
```shell
kubectl edit deployment nginx-depl
```

### kubectl get node
* Displays all the nodes in the cluster

### kubectl apply -f [file name]
* Manages applications through files defining K8s resources
* Creates a new Deployment if it doesn't exist or update the configuration if already exists
* `--namespace=[namespace_name]` can be used to specify the namespace where the resource should be created 
* Examples:
```shell
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml
```

### kubectl get all
* Displays all the pods created in the cluster

### kubectl get pod | configmap | secret | service or svc | deployment | replicaset | namespace | endpoints | ...
* Display different resources like pods, configmaps, secrets, service, and more.
* `-o wide` can be used to get wide or longer output.
* `-o yaml` can be used to get yaml output.
* `-o yaml > [filename]` can be used to get yaml output into a file.
* `--watch` can be used to keep watching the status change of the resource
* `-n [namespace_name]` can be used to list resources from specific namespace. If it is now provided the resources are listed from default namespace.
* Examples:
```shell
kubectl get pod
kubectl get configmap
kubectl get secret
kubectl get service
kubectl get svc
kubectl get deployment
kubectl get replicaset
kubectl get namespace
kubectl get endpoints

kubectl get pod -o wide
kubectl get node -o wide

kubectl get deployment nginx-deployment -o yaml

kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml

kubectl get pod --watch

kubectl get configmap -n my-namespace
```

### kubectl --help
* Displays all the sub commands we can use with kubectl

### kubectl get --help
* Displays how we can use get command with kubectl

### kubectl describe [resource_type] [resource_name]
* Displays detailed information
* `-n [namespace_name]` can be used to specify the namespace for the resource
* Examples:
```shell
kubectl describe service webapp-service
kubectl describe pod webapp-deployment-dcffd6bcc-gsmlt

kubectl describe ingress dashboard-ingress -n kubernetes-dashboard
```

### kubectl logs [pod_name]
* Display logs of container
* `-f` can be used to stream the logs.
* Example:
```shell
kubectl logs webapp-deployment-dcffd6bcc-gsmlt
kubectl logs webapp-deployment-dcffd6bcc-gsmlt -f
```

### kubectl exec -it [pod_name] -- bin/bash
* Starts interactive terminal of the application container
* Example:
```shell
kubectl exec -it mongo-depl-887485654-sjq96 -- bin/bash
```

### kubectl delete [resource_type] [resource_name]
* Deletes the deployment with specified name
* Example:
```shell
kubectl delete deployment mongodb-deployment
```

### kubectl create namespace [namespace_name]
* Creates a new namespace
* Example:
```shell
kubectl create namespace my-namespace
```

### Note: Demo project is attached with the repository.

## How to base64 encode value for Secret
```shell
echo -n 'plaintext' | base64
```

## References
* Video 1: https://www.youtube.com/watch?v=s_o8dwzRlu4
* Video 2: https://www.youtube.com/watch?v=X48VuDVv0do
* Mongo DB Container: https://hub.docker.com/_/mongo
* Mongo Express Container: https://hub.docker.com/_/mongo-express
* Web App Container: https://hub.docker.com/r/nanajanashia/k8s-demo-app