                            
  # KUBERNETES:
 

 ## K8S ARCHITECTURE:
   
  
### ControlPlane: MasterNode
     
  + 1. **ApiServer:**
    + Primary mgnt component/ exposes k8s API serving as frontend interface for all cluster operations
          handles restful Api from kubelet
  + When you run a kubectl command, it communicates with the kube API and then it gets the data from the ETCD
  + The request is first authenticated and then validated. 
  + 2. **ETCD:**
    + It is a key-value store, It stores the cluster's configuration data,
  + 2 **Scheduler:**
    + Responsible for making decisions about pod placement on worker nodes in the cluster.
    + It examines resource requirements, quality-of-service constraints, 
      affinity, anti-affinity, and other policies to determine the most suitable node for running a pod.
      it doesn't place resources on nodes but makes the decision
  + 3. **ControllerManagers:**
   + Manages Node lifecycle, desired pod number, and services it continuously monitors the state of resources in
       the cluster and ensures that they match the desired state.
   + Node Controler: monitors the status of the nodes every 5sec. it waits for 40 secs and if unreachable, it evicts
      the pods running on the node.
  + ## ReplicationController:
 + it monitors the status of the replica set and makes sure the desired states are maintained.

         ReplicaSet
         DaemonSet
         ReplicationController
 ## WorkerNodes:
 
  + 1. **kubelet**: 
     responsible for managing and operating containers. communicate with control-plane 
  it registers nodes into the cluster. It monitors nodes and pods in the cluster every 10 minutes and 
  relates feedback to the API which is stored in the etcd.cluster
  + 2. **container runtime**:
   [Container-d] docker pulling containered images
  + 3. **kube-proxy**: 
    enables network communication and load balancing between pods and services within the cluster.
  every pod in a cluster can communicate with other pods in the cluster by using IP address of the pod.
  to access each pod, a service has to be created and then you can access the pod by using the service name.
  the service is not an object, the kube-proxy creates rules that allow traffic routing within the cluster.

         ClusterIP
         NodePort
         LoadBalancer
+ kubernetes-client:
```bash
  kubectl  
      kubectl create/delete/get/describe/apply/run/expose 
``` 
      kubeconfig [.kube/config ] file will authenticate 
                                 the caller admin/Developer/Engineer 

In Kubernetes Quality of Service (QoS) refers to the priority and resource guarantees provided to different 
workloads or pods within a cluster. Kubernetes provides mechanisms to manage QoS to ensure that critical 
workloads receive the necessary resources and performance, while also allowing for efficient resource 
utilization.

## PODS:
- The aim is to deploy applications as containers running on a set of machines. 
- Containers do not run directly on the node but on Pods. 
- Pod is the single instance of an application and the smallest object in k8s/.
- If the user base increases, you can scale additional pods on the node, and if the node runs out of storage,
  you can spin up new nodes and assign new pods of the same or diff containers to it.
- Two containers of the same kind can not run in the same pod.
- There are multi-container pods which are helper containers running a process for the pod. they both live and die 
at the same time. they can refer to each other using localhost.
```bash
kubectl run nginx --image nginx
```
it creates a pod call nginx and also pulls the image nginx from a public docker repo 

apiVersion: # this is the version of the k8s API, it is mandatory Pod: v1 , service: v1 
#replicaSet: apps/v1 , Deployment: apps/v1. It is also a string value 
kind: # This refers to the type of object to be created such as Pod, ReplicaSet, Deployment, etc string
metadata: # This is data about the object such as name and labels. Metadata is a dictionary, it is indented
   name: myapp #
   labels: # it is a dictionary and can take any kind of key-value pair such as
      app: myapp # It is a string
      type: front-end
note: you can only add name and labels under metadata or specification from k8s 
spec: # this provides additional information about the object to create. it varries per object
  containers:  list/array
      - name: nginx-container  # first item in the list
        image: nginx
      - name:
        image:

+  example:
```sh
apiVersion: v1
kind: Pod
metadata:
  name: myapp 
  labels:
    app: myapp
    type: front-end
spec:
  containers:  
  - name: nginx-container
    image: nginx

```
```sh
- kubectl apply/create -f <filename> #to create declaratively from a yml file
- kubectl get/describe pods <podname> #to get the pod spec
- kubectl create deployment redis-deployment --image=redis123 --dry-run=client -o yaml > deployment.yaml
- kubectl create deployment redis-deployment --image=redis123 -o yaml #print out output
- kubectl edit pod <podname>
```
## REPLICASETS:
Controllers are the brain behind k8s, they monitor k8s objects and respond accordingly.
- the replication controller helps increase the number of pods in the node for high availability.
- It also serves as recreating a pod if it fails.
- It creates pods across nodes to balance the load
- replication controller is replaced by replicasets
- it maintains the desired number of pods specified in your object definition file 
```sh
apiVersion: v1
kind: ReplicationController    #DEPRICATED
metadata:
  name: myapp-rc
      labels:
        app: myapp
        type: fe
spec:
  template: # Here you provide a pod template which is intended to be managed by the replicationcontroller
    metadata:
        name: myapp 
        labels:
          app: myapp
          type: front-end
    spec:
       container:
       - name: nginx-container
         image: nginx 
  replicas: 3
```
```sh
- kubectl create -f <filename>
- kubectl get rc 
```
#replicasets requires a selector field | it is not a must
#It helps the replica set defines what pods fall under it although pod spec has already been mentioned in the spec
#This is because it can manage pods that were not created to be managed by the rs
```sh
apiVersion: apps/v1
kind: ReplicaSet  # <-- Specify the correct kind
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        type: front-end  # This must match the label that was input in the object metadata section
    spec:
      containers:
      - name: nginx-container
        image: nginx

   
```sh
k get rs 
k get pods 
```
## Labels and Selectors:
+ Labels are used as filters for ReplicaSet. Labels allow the rs to know what pod in the cluster or nodes 
placed under its management since there could be multiple pods running in the cluster.
+ The template definition section is required in every rs, even for pods that were created before the rs 
+ This is due to the fact that if the pod fails and is to be recreated, it will need the spec to recreate it

- if you want to scale from 3 to 6 replicas, update the replicas to 6 and run
```sh
 kubectl replace -f <filename>
 kubectl scale --replicas=6 <filename>
 kubectl scale -- replicas replicaset name 
 kubectl edit pod/rs/rc/deploy <podname>
```
## Deployments

+ When you want to deploy an application, you may want to deploy several pods of that application for high 
availability. When a newer version of that application is available in docker, you want to gradually update
to avoid downtime of the application.
suppose an update has issues, you will want to do a rollback to the previous working version   
+ You can also make changes such as resources and the number of pods.
+ Deployment provides the capability to upgrade the underlying instance such as rolling-update, pause, upgrade

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        type: front-end # This must match the label that was input in the object metadata section
    spec:
      containers:
      - name: nginx-container
        image: nginx

    
```sh
kubectl get deployment
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

## Services
Kubernetes service enables communication between various components within and outside the application and between  
other applications and users. Services enable the frontend app to be available to users and btw frontend and backend

For external communication,

the Kubernetes node has an IP, the host OS which is in the same network has an IP (private), and the pod has an ip but on  
a separate network
To access the application externally, the k8s service enables communication from pods on nodes

## TYPES:

### 1. *NodePort*:
The k8s service maps a port on the Node to a port on the Pod(target)
The NodePort is a port range on the Node that gives external access. it ranges from 30000-32767

apiVersion: v1
Kind: Service
metadata: 
  name: myapp-svc
spec:
  type: NodePort
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. This is a mandatory field
      nodePort: 30008  #external node port on service 30000-32767. a random port will be allocated if not specified
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: front-end

kubectl create -f <filename>
kubectl get svc 
curl IP:30008

- In the case of multiple pods running the same application, you need to maintain the labels and selector section with
the same values. the service uses a random algorithm to route traffic to all pods with that same label.
- If the pods are running on different nodes in the cluster, you can access it by calling the IP of any node in the  
cluster. Service is a cluster-wide resource in k8s.

## 2. *ClusterIP*:

  A full-stack web app typically has a number of pods such as frontend pods hosting a web server,
  the backend hosting the app, and pods hosting a db. Kubernetes service can group all pod groups together 
  and provide a single backend to access the pods. you can create another service for all pods running the DB 
  These pods for different applications can therefore be scaled like microservices without impacting the other.
  A separate SVC for frontend, for backend, and for db. 
  - This type of service that allows communication between pods in a cluster is called cluster IP service.
```sh
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
  ports:
    - protocol: TCP
      port: 80  # (port on Pod). it will assume port if not specified
      targetPort: 80  # port on service. this is a mandatory field
  type: ClusterIP

 
```
kubectl create -f <filename>
kubectl get svc 

the service can be accessed by other pods in the cluster using the service name or ClusterIP

### 3. *LoadBalancer*:
When multiple pods of an application are deployed, they can all be accessed by using the diff IPs of the nodes
mapped to the nodePort. 
However, end users need to be provided with a single endpoint that can route traffic to all the pods.
K8s have native support for cloud platforms 
```sh
apiVersion: v1
kind: Service
metadata: 
  name: backend
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: backend

```
## NAMESPACES:

  A namespace is simply a distinct working area in k8s where a defined set of resources rules and users can  
  be assigned to a namespace. 
- By default, a k8s cluster comes with a default namespace. Here, a user can provision resources.
- the subsystem namespace is also created by default for a set of pods and services for its functioning.
- The kubepublic is also created to host resources that are made available to the public.
- Within a cluster, you can create diff namespaces for different projects and allocate resources to that namespace.
- Resources from the same namespaces can refer to each other by their names,
- they can also communicate with resources from another namespace by their names and append their namespace.
eg msql.connect("db-service.dev.svc.cluster.local")

kubectl get pods > will list only pods in the default namespace
kubectl get pods --namespace=kubesystem

kubectl apply -f <filename>  ==> will create object in the default namespace
kubectl create -f <filename> --namespace=kubesystem  ==> will create object in the kubesystem namespace

To ensure that your resources are always created in a specific namespace, add the namespace block in the resources
definition file
```sh
apiVersion: v1
Kind: Service
metadata: 
  name: backend
  namespace: dev  # This resource will always be created in the dev namespace, and will create the ns if it didn't exist
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. this is a mandatory field
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: backend
```

```sh
apiVersion: v1
Kind: NameSpace
metadata:
  name: dev
```
OR 
 kubectl create namespace dev
 kubectl get ns 

to set a namespace as the default namespace so that you don't always have to pass the NameSpace command, you need to set
set the namespace in the current context

kubectl config set-context $(kubectl config current-context) --namespace=dev
contexts are used to manage all resources in clusters from a single system 

To view resources in all NameSpaces use the 
kubectl get pods --all-namespaces

### Resource Quota:
To set a limit for resources in a namespace, create a resource-quota object in
```sh
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi

```
kubectl apply -f <filename>

### Declarative and Imperative :
  these are different approaches to creating and managing infrastructure in IaC.
- The Imperative approach is giving just the required infrastructure and the resource figures out the steps involved.
eg kubectl create deployment nginx --image nginx
    kubectl set image deployment nginx nginx=nginx:1.18
    kubectl replace -f nginx.yaml

    it creates resources quickly but is difficult to edit or understand what is involved.

- in the Declarative approach, a step-by-step approach through writing configuration files
   Here we can write configuration files 
   Here, changes can be made as well as the resources can be versioned.
   resources can also be edited in the running state using the kubectl edit command
   The best practice is always to edit the configuration file and run the replace command rather than using the edit command.

   when you use the kubectl apply command, it will create objects that do not exist, and when you want to update the object,
   edit the yml file and run kubectl apply again and the object will pick up the latest changes in the file.
```sh
kubectl run --image=nginx nginx 
kubectl create deployment --image=nginx nginx 
kubectl edit deployment nginx 
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginxv
```
```sh
kubectl run nginx --image=nginx --dry-run=client -o yaml    > nginx-deployment.yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
k create deploy redis-deploy --image=redis --replicas=2 --namespace=dev-ns
kubectl run httpd --image=httpd:alpine --port=80 --expose   #will create pod and service
```
when you run a kubectl apply command, if the object stated in the file does not exist, it is created.
another live object configuration is created with additional fields and can be viewed using the  
- kubectl edit/describe object <objectName>
there is the last applied file that provides details about the last image of the live configuration.


SCHEDULING:
============

- there is a builtin scheduler in the cluster control-plane, that scans through nodes in the cluster and schedules
  pods on nodes based on several factors such as resources,
- But if you want to override and schedule your pods on specific nodes for some reason, you can do that by
  specifying the nodeName in the pod definition file.
- If a scheduler does not exist in the cluster, the pod will continually be in a pending state.
- If you need a pod to run on a specific node, declare it at the time of creation.
- Kubernetes does not allow node modification after the pod has already been created.
- It can only be modified by creating a binding object setting the target to the NodeName and then sending a post 
 request to the pod's binding API.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: controlplane
```
RESOURCE REQUIREMENTS:
=======================
- every pod requires a set of resources to run.
when a pod is placed on a node, it consumes the resources on that node.
the scheduler determines the node a pod will be scheduled on based on resource availability.
if nodes have insufficient resources, the scheduler keeps the pod in a pending state.
- You can specify the resource requested by  a pod to run.
the scheduler will look for a node that has that resource specification and place that pod on it.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "4Gi"
        cpu: 2
      limits:
        memory: 
        cpu: 
  nodeName: controlplane
```

- When a pod tries to exceed resources out of its limits, the system throttles the containers so that it doesn't
  use more than its limit,
- as for memory, a container can use more memory resources than its limit. but a pod cannot
- By default, k8s does not have a request and limit set, therefore resources can consume as much as they need.
- One pod can consume more and prevent others from running.
- When a CPU limit is set without request, k8s sets the request to the same as the limit
- When CPU requests and limits are set, then they stay within the range. But if one pod isn't consuming resources,
  then it is securing resources that other pods could use.
- When requests are set without limits, any pods can consume as many CPUs are required and when a pod needs more  
  resources, it has a guaranteed resource but without a limit. Make sure all pods have requests set.

LimitRanges as objects can be used to ensure that every pod created has some default values at the namespace level.
You can set it for both CPU and memory at the ns level. all pods will assume that standard.
ResourceQuota can also be used to set resource limits at the level of the NameSpace.

## QUALITY OF SERVICE IN KUBERNETES:

Kubernetes offers three levels of Quality of Service:

+ 1. **BestEffort:**
   - Pods with BestEffort QoS are not guaranteed any specific amount of resources.
   - They are scheduled onto nodes based on availability, and they can use whatever resources are available at that time.
   - These pods are the first to be evicted if resources become scarce.

+ 2. **Burstable:**
   - Pods with Burstable QoS are guaranteed a minimum amount of CPU and memory.
   - These pods can burst beyond their guaranteed minimum if the resources are available.
   - If other pods on the node need resources, Burstable QoS pods might be limited in their burst capacity.

+ 3. **Guaranteed:**
   - Pods with Guaranteed QoS are guaranteed a specific amount of CPU and memory.
   - These pods are not allowed to exceed the resources they have been allocated.
   - Kubernetes tries to ensure that nodes have enough available resources to meet the guaranteed requirements.

Kubernetes determines the QoS level of pods based on the resource requests and limits specified in the pod's configuration:

- **Resource Requests:** The minimum amount of resources a pod requires to run. 
    These requests are used by the scheduler to make placement decisions.
- **Resource Limits:** The maximum amount of resources a pod is allowed to use.
    Exceeding these limits could lead to throttling or pod termination.

Kubernetes uses the relationship between requests and limits to categorize pods into different QoS classes. 
The actual QoS class assigned to a pod depends on how its requests and limits are set:

- **BestEffort:** Pods with no resource requests or limits.
- **Burstable:** Pods with resource requests, but without memory limits or with memory limits lower than their requests.
- **Guaranteed:** Pods with both CPU and memory limits set to be higher than or equal to their resource requests.

Setting appropriate resource requests and limits for pods is crucial for efficient resource allocation and QoS management 
within a Kubernetes cluster. Properly configured QoS levels help ensure that critical workloads are prioritized 
and that the cluster operates smoothly without resource contention issues.

## DEAMONSETS:

Deamonsets are like replicasets which help you run one instance of pods, but it runs one copy of your pod on every  
node on the cluster.
the deamonset ensures that one copy of the pod is always running on every node in the cluster.
A use case is if you are deploying a log collecting or monitoring agent.
objects like the kube-proxy and network use deamonsets because they have to run on every node.
```sh
apiVersion: apps/v1
kind: Deamonset
metadata:
  name: monitoring-agent
spec:
  selector: monitoring-agent
  matchLabels:
    app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - image: monitoring-agent
        name: monitoring-agent
```
```sh
k create -f <filename>
kubectl get deamonsets 
kubectl describe deamonsets
kubectl get daemonsets --all-namespaces 
```
- How do you get pods to be scheduled on every node?
- one approach is to use the node name to bypass the scheduler and place a pod on a desired node.


STATIC PODS:
============
- Without the controlplane which contains the API server, you can store your configuration files at the path 
  /etc/kubernetes/manifest,.
- kubernetes frequently visit this directory and any manifest file here to create pods will create the pod.
- it can create only pods, another object will need the controlplane
- The static pod folder can be any directory, but the path is set in the kubelet.service file in the kubeconfig.yaml
- Static pods can be viewed by running the docker ps command.
- The kubelet can create resources in k8s either by using configuration files or by listening to the kube API endpoints
  from the control controlplane.
- If you run the k get pods command, it will also list the static pods. This is blc a mirror of the static pods that are  
- created in the kubeapi but like other pods, can not be edited, except in the pod definition files.
- A use case for static pods is when you want to install components of the k8s control plane on every node, then you start 
  by installing the kubelet service and then create pod definition files that use docker images of all other 
  components of the controlplane and place them in the etc/kubernetes/manifest dir

kubectl get pods -n kube-system

Multiple Schedulers:
====================
You can deploy an optional scheduler added to the custom scheduler and configure it to schedule specific pods.
you can use a pod definition file or wget the scheduler binary and remane the scheduler to a different name.
to make sure that your object to be created is managed by that scheduler, you can add a scheduler option under
the spec section and pass the name of the scheduler.

kubectl logs object objectname

scheduling queu ===> priority sort
filtering       ===> NodeName/NodeUnschedulable/NodeReourceFit
scoring         ===> NodeReourceFit/ImageLocality
binding         ===> Defaultbinding

LOGGING AND MONITORING:
=======================
We can monitor the applications deployed in k8s as well as the Kubernetes cluster.
to monitor resources in the cluster, we can monitor
- node level metrics  #number of nodes, healthy, memory performance, CPU utilization
- pod level metric # number of pods and their CPU utilization

Kubernetes by default does not come with any monitoring agent but metric servers can be used and other resources  
such as Prometheus.
- You can have one metric server per cluster, the metric server retrieves metrics from pods and nodes  
 and stores them in memory. It does not store the metric in the disk so you can not store see historical metric.

You can clone the metric server from the github repo and run it. 
cluster performance can be seen by running  

kubectl top node 
kubectl top pods

managing application logs: 
  When you run a container in a pod, you can stream the logs by running the  
   kubectl logs -f <podname>
in the case of multi-container pods, you need to specify the name of the container individually.

APPLICATION LIFECYCLE MANAGEMENT:
=================================
## 1. Rolling Update and rollback:
  when you first create a deployment, it triggers a rollout which can be rev 1
  later when the image is updated, a new rollout is made called rev2
  to see the status of the rollout, run the  
  kubectl rollout status deployment/<deploymentname>

  to see the revision and the rollout history, run the
   kubectl rollout history deployment/myapp-deployment

there are two types of deployment strategies.

## 2. RECREATE:
- You can delete the existing deployment and then make a new deployment with the newer version
- This will lead to app downtime. this is not the k8s default strategy.
  
## 3. ROLLING UPDATE:
- Here, pods replicas are progressively destroyed and replaced with newer pods to ensure there is no downtime 
- This is the k8s default update strategy.

update can be done by changing the version of the image, replicas  
kubectl apply -f <filename>

it is advisable to manually edit the definition file than using the imperative approach because with an Imperative,
changes will not be saved in the definition file.

to rollback run the kubectl rollout undo deployment/myapp-deployment
```sh
kubectl apply -f deployment
kubectl get deployment
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/app 
```
ConfigMaps:
===========
this is a way of managing environmental variables in k8s. you can manually inject this variable by passing them  
as env. But with many def files that require this variable, then you need to create them as a separate object in k8s 
and simply reference them in your object definition file. This can be done using ConfigMaps and Secrets.

ConfigMaps are used to pass configuration data in the form of key-value pairs in k8s and then injected into pods.
```sh
kubectl create configmap <ConfigName> --from-literal=APP_COLOR=blue \
                                   --from-literal=APP_MODE=prod  
                   OR 
kubectl create configmap app-config --from-file=<pathtofile>
```
```sh
APP_COLOR: blue 
APP_MODE: prod

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue 
  APP_MODE: prod
```
kubectl get configmaps


to inject the  env to the running container, add the envFrom section under the spec section  
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
        name: app-config
```
to create resource, use kubectl create -f <filename>

You can also ref a single env from a configmap 

env:
  - name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR

You can as well inject it as a volume

volumes:
- name: app-config-volume
  ConfigMap:
    name: app-config

SECRETS:
========
Secrets just like configmaps are used to store configuration data which can be injected into an object in k8s.
Unlike configmaps, secrets store sensitive data such as passwords and keys in an encoded manner.
You can create a secret imperatively by using:
```sh
  kubectl create secret generic <secretName> app-secret --from-literal=DB_Host=mysql   \
                                                        --from-literal=DB_User=root
```
    You can ref the secret from a file using the --from-file=app-secret
    For a declarative approach
```sh
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root 
  DB_Password: password
```
It is however not advisable to pass your secrets in plain text as if defeats the entire purpose.
To convert the data from plaintext to an encoded format, on a Linux system, use the  
```sh 
echo -n 'mysql' | base64
echo -n 'root' | base64
echo -n 'password' | base64
```
```sh
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: sjhdvdv=
  DB_User: fnvjsf== 
  DB_Password: sffvnhri
```
copy the corresponding encoded values and replace and inject them into the file.

kubectl get secrets app-secret
kubectl describe secrets
kubectl get secrets app-secret -o yaml

to decode encoded values use the  
```sh
echo -n 'djvfjdo=' | base64 --decode
```
to inject encoded values into the pod object, use the 
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
        name: app-secret
```
Secrets are not encrypted but rather encoded and can be decoded using the same method. Therefore, do not upload your
secret files to the GitHub repo.

You can enable encryption at rest:
```sh
kubectl get secrets --all-namespaces -o jason | kubectl replace -f -
```
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

Multi Container PODS:
=====================
the idea of decoupling a large mom=nolithic application into small components called microservices,
allows us to deploy a set of small independent and reusable code. This setup allows us to manage, or update only,
small portions of the app instead of the entire app. It might require running two apps or components in the same  
container. An example is a web server and a log agent deployed in the same container. They share the same lifecycle,
they are created and destroyed together, and they share the same network and the same volume of resources.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: webapp
spec:
  containers:
  - image: nginx     # container1
    name: nginx
    ports:
      - containerPort: 8080
  - image: log-agent     #container2
    name: log-agent
```
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
    
to exec into a container in kubernetes, run the 
```sh
kubectl -n <Namespace> exec -it <ContainerName> -- cat /log/app.log
```
### InitContainers:

  these are also sidecar containers just like in a multicontainer pod. But they do not run constantly like the multi-containers
  Init containers are designed to run a particular process and then once the process is complete, they are exited.
  they may provide a service or run a script that starts a process in the main container and then exits.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```
CLUSTER MAINTENANCE:
====================
This is important to know how and when to upgrade a cluster, and how to understand disaster recovery.
### 1. OS UPGRADE:
  To take down nodes in the cluster for updates or security patches on the node. When a node hosting the pods goes down,
  all the pods will not be accessible to users. Except there were replicas of that pod on another node.
  If the node lasts less than 5 minutes, the pods can be rescheduled, but if it exceeds 5 minutes, the controller will  
  consider it dead. If the pods were part of a ReplicaSet, they are recreated on other nodes depending on the nodeAffinity policies

  A quick update can be done when you are sure it will last less than 5 minutes, and if the pods on that node are part of a ReplicaSet.
  this will ensure that the application remains accessible.
  To safely do an upgrade on the nodes, you can drain the node for
```sh
  kubectl drain node-1 # --ignore-daemonsets 
  kubectl cordon node-2  # to make node unschedulable
  kubectl uncordon node-2 
```
  The node will be marked as unschedulable and pods will be gracefully terminated and recreated on other nodes.
  You can, path the nodes and make them available. You need to manually uncordon the node to make it schedulable.

  After a node is uncordon, it will require that pods are scheduled on it afresh, pods that were evicted during the drain 
  will not be automatically rescheduled.

  If a pod is not part of a ReplicaSet on a node, k8s internal security will not permit that node to be drained unless
  you manually delete the pod.
  nevertheless, you can use  --force flag to force delete the pod. This will permanently delete the pod on that node  
  and will not recreate it on another node because it was not part of a ReplicaSet.

### 2. Cluster Upgrade:
  Kubernetes is released in version and there are minor versions such as the alpha and beta versions before a more stable 
  release is made.
  None of the cluster components can be of a version higher than the API Server, except for the kubelet service.
  You can upgrade component by component.
  At any time, k8s supports only the latest three minor versions. It is good to upgrade your cluster before the version is unsupported.
  You should upgrade only one version higher at a time and not to the latest version if you were not using the previous one.

the upgrade depends on how the cluster is set up. between managed and self-managed clusters. managed clusters provided by the cloud is easier.
the cluster is being upgraded component by component and while the master node is being upgraded, the Kube API, and controllers,  
go down briefly. this does not affect the worker nodes.

To upgrade the worker nodes, if you do all of them at once, users will not be able to access the app. 
You can also upgrade one node at a time, which guarantees your pods are not all down. Or u use new nodes with newer
software and then move all pods?


https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
```sh
cat /etc/*release*  # to see the OS the nodes are using
kubeadm upgrade plan # to all the latest and stable versions
k drain controlplane --ignore-daemonsets
apt update
apt-cache madison kubeadm  # select the kubeadm version
apt-get upgrade -y kubeadm=1.12.0-00  # It has to be upgraded before the cluster components
```

to upgrade the cluster, use the 
```sh
kubeadm upgrade apply v1.12.0
```
the next step is to upgrade the kubelet.
the kubelet service must be upgraded manually
```sh
apt-get upgrade kubelete=v1.12.0-00
systemctl restart kubelet.service
```
### 3. Node Upgrade:

  https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/

  You need to first move all the workloads from the node to other nodes using the 
  ```sh
  kubectl drain node01 # This makes the node unschedulable
  apt-get upgrade -y kubeadm=v1.12.0-00
  apt-get upgrade -y kubelete=1.12.0-00
  kubeadm upgrade node config --kubelet-version v1.12.0
  systemctl restart kubelet
  kubectl uncordon node01
```

  perform the same steps to upgrade all other worker nodes.

  ssh <nodename>

BACKUP AND RESTORE:
===================
With respect to resources, declarative approaches can be used to save your configuration files. A good practice is to save
these codes in a source code repo like GitHub. But if an object is created imperatively, it will be difficult to keep track.
therefore, the KubeAPI server is a place to get all created resources.
All resource configurations are saved in the kube-apiserver

1. kubectl get all --all-namespaces -o yaml > all-deploy.yaml

there are tools like VELERO that can help in taking backups of the cluster.

The ETCD stores information about the state of the cluster.
So you can choose to backup the etcd cluster itself. it is found in the controlplane
data is stored in the data directory.
it also comes with the built-in snapshot utility

etcdctl snapshot save snapshot.db
etcdctl snapshot status snapshot.db


A snapshot directory is created. To restore the cluster from this backup, stop the kubeapi server and run the  

etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
systemctl daemon-reload
service kube-apiserver start

k logs etcd-controlplane -n kube-system | grep -i 'etcd-version'


ls /etc/kubernetes/manifest
k edit etcd.yaml
export ETCDCTL_API=3

ETCDCTL_API=3 etcdctl snapshot save --endpoint= \
--cacert= \
--cert= \
--key= \
/opt/snapshot-pre-boot.db  #location to save backup

to restore the original state of the cluster using the backup file, you can use the etcd restore <filename>


etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db 

ls /var/lib/etcd-from-backup

vi /etc/kubernetes/manifest/etcd.yaml

edit the Hostpath and add /var/etcd-from-backup

