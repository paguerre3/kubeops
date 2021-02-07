# kubeops
kubernetes guide that serves as theorical and practice documentation of core concepts


---
### intro to k8s
- what is kubernetes?
- k8s architecture
- main k8s concepts
- minikube and kubectl - localsetup (dockerdesktop with kubernetes?)
- main kubctl commands - k8s cli
- k8s yamel configuration file
- hands-on-demo
- k8s namespaces - organize your components
- k8s ingress
- helm - package manager
- volumes - perist data in k8s
- k8s statefulset - deploy stateful apps
- k8s services


---
### what is kubernetes?
- open source container orchestration tool
- in its foundation manages containers that could belong to docker or other technologies
- it helps managing containerized applications, e.g. 100 or 1000, in different deployment environments, i.e. assisting the management of containerized applications in physical, virtual, cloud or hybrid environments
- the trend from monolith to microservices increses the usage of containers, i.e. more artifacts to deploy are containerized as microservices have smaller tasks. Managing the deployment of projects/applications built on a large amount of microservices containerized using configuration scripts and tools is really problematic so orchestration technologies as k8s help handling this problem

**NOTE**
> ochestration tools offer high availability or no downtime, scalability as a way for improving performance, and disaster recovery i.e. backup and restore


---
### kubernetes core components
- NODE or worker node is a simple server that contains one or more pods in it, e.g. node1=physical or virtal machine. Physical machine layers=hardware+os kernel+application vs virtual machine=os kernel+application vs virtual container=works at application layer as its faster because runs on top of host os kernel and smaller, e.g. being os kernel=linux and application=alpine or ubuntu
- POD is an abstraction over a container considered as the smallest unit of k8s, e.g. pod1=abstraction-over-docker-container-of-java-app and pod2=abstraction-over-docker-container-of-mongo-db inside node1/server. A pod creates a running environment on top of a container because k8s abstracts the running environment of the container technology so it isn't coupled to an scpecific technology as docker/docker-runtime, i.e. you only interact with kubernetes layer and the underlaying container technology can be replaced if needed

**NOTE**
> Usually 1 containerized application per pod
- k8s offers an out of the box virtual network which means that each pod inside a node has its own IP, i.e. each Pod gets its own IP address (not the container) so each pod communicates with another pod inside the same node using the internal IP address (not external IP address)
<img src="https://github.com/paguerre3/kubeops/blob/main/support/1-pod-communication-ip.PNG" width="23%" height="30%">

- pods in k8s are ephemeral, i.e. that they can die because of a crash and when it happens a pod gets "recreated" and a "new" IP is assigned (this is an inconvenient as communication is lost between pods of the same node). To resolve the situation there is another component in k8s called Service
- SERVICE is a permament IP address that can be attached to each pod. The lifecycle of Pod and Service are not connected, i.e. that even if a pod dies the Service and its IP address will stay, e.g. pod1 will still communicate with the "new" pod2 after recreation. "External" Service is being used to expose communication for an external browser/location, e.g. external ip for a public rest-api containerized. "Internal" Service is being used to avoid exposing communication to the external world, e.g. internal ip of a database containerized
<img src="https://github.com/paguerre3/kubeops/blob/main/support/2-pod-communication-svc.PNG" width="48%" height="30%"> 

- External type of Service exposes the IP of the Node which isn't practical, e.g. http://ip:port. A better solution is exposing the domain name using a secured protocol, e.g. https://my-app.com
- INGRESS is the component resposible for forwardng the communication between the domain of the appication and the service that holds the permanent ip address of the containerized application. In other words, it routes the traffic
<img src="https://github.com/paguerre3/kubeops/blob/main/support/3-pod-communication-ingress.PNG" width="48%" height="30%">

- usually containerized application configurations are built within the application as properties, e.g. data base URL usually specified inside properties of the custom application that comunicates with the data base, i.e. if there is a change of the data base endpoint a rebuild/new-push-into-repo/new-pull-from-repo of the application image that uses it is required+restart

**NOTE**
> The case of exposing Dockerfile/entrypoint/env variables accessible in docker-compose doesn't require image rebuild of the compose after changes but it works in a lower level of abstraction i.e docker-runtime and it needs restart, e.g. docker-compose -f {docker-compose.yml} down/up, e.g. [Dockerfile](https://github.com/paguerre3/dockerops/blob/main/Dockerfile) + [docker-compose](https://github.com/paguerre3/dockerops/blob/main/complete.yml)
- CONFIG-MAP in k8s is the the external configuration component of application/s containerized that can be shared among each pod, e.g. DB_URL=mongo-database, so if a change of "value" is requested there isn't any need of doing a rebuild on the application image/s that is/are using the common "key", i.e. DB_URL key remains the same at image level while the ConfigMap has its "value" updated=mongo-db. Putting credentials in ConfigMap is insecure although they are also considered external configuration
- SECRET is an external configuration designed to store data in a secured manner with the same benefits of ConfigMap, e.g. to save user and password of data base or certificates. Secret stores contents in base64 encoded while in ConfigMap is simple text. The build-in security mechanism isn't enabled by default
<img src="https://github.com/paguerre3/kubeops/blob/main/support/4-pod-communication-cm-secret.PNG" width="43%" height="30%">

**NOTE**
> inside a Pod values from ConfigMap and Secret can be seen using environment variables or as a properties file 
- because a pod is ephemeral if it holds a stateful application containerized, e.g. a data base, when the pod gets recreated the data is gone if no volume is mounted
- VOLUMES attaches a physical storage on a hard drive to a Pod so when pod gets recreated the data isn't lost. Storage can be on a local machine i.e. on the same server Node where the pod is located or in a remote storage outside of k8s cluster e.g. a cloud storage or custom onpremise storage
<img src="https://github.com/paguerre3/kubeops/blob/main/support/5-pod-communication-volumes.PNG" width="23%" height="30%">

**NOTE**
> its important to understand the difference between k8s cluster and a storage regardless the type, e.g. local or remote. Think of storage as an "external" hard drive plugged into a k8s cluster because kubernetes cluster doesn't manage data persistance! i.e. that k8s users are responsible for handling backup and restore of the data
- in one server node deployment if one pod crashes and it was the one being exposed in pubic then the external client that was using it will experience downtime i.e. the user can't reach the application containerized inside the pod that is down. The advantave of distributed systems in containers as k8s is that instead of relying in one server node it Replicates everything in multiple server nodes so the load is distributed accomplishing high availability and no downtime, e.g. a replica of pod1 that belongs to node1 is created inside node2 and this replica pod2 is connected to the same Service that acts as a LoadBalancer with permanent IP

**NOTE**
> Service is a persistent static/permanent ip address with a dns name to avoid adjusting the endpoint when a pod dies. Service is also a LoadBalancer which means that it will receive the request and forward it to the pod that is less bussy
- in k8s instead of creating the replica for the 2nd pod, the user defines Blueprints for pods where its specified how many replicas per each pod are necessary
- DEPLOYMENT is the blueprint of "stateLESS" application pods where Replica requirements are defined. In practice, k8s users don't create pods and instead they create Deployments because replica specification and scale up/down mechanism are being done with the usage of it

**NOTE**
> pod is an abstraction layer on top of a container and a Deployment is considered an abstraction layer on top of the pod that is more convenient for k8s cluster management purposes, e.g. it helps when replicating and scaling pods
<img src="https://github.com/paguerre3/kubeops/blob/main/support/6-pod-communication-svc-deployment.PNG" width="48%" height="30%">

- Deployments can't be used in case of pods with "stateFUL" applications containerized, e.g. data bases, because during pod replication they will be be replicating/clonning data therefore data inconsistencies by "how" the external-storage-system handles synchronization might occur, e.g. there is no mechanism defined inside a Deployment that specifies which pods are writters and which ones are readers of the storage system
- STATEFULSET is meant for managing stateful applications in k8s cluster, e.g. mongodb, mysql and elastic-search. SatefulSet is  resposible of pod replication and ensuring that data inconsistencies don't occur with the external storage system attached to k8s cluster, e.g. defining which pods are writters and which ones are readers
<img src="https://github.com/paguerre3/kubeops/blob/main/support/7-pod-communication-svc-statefulset.PNG" width="48%" height="30%">

**NOTE**
> deploying pods of stateful containerized applications with StatefulSet isn't easy so data bases are ofter hosted outside k8s cluster and pods of stateless containerized applications communicate with external storages directly without even using pods of stateful applications and volumes service offered


---
### kubernetes architecture
- NODE is the worker machine in k8s cluster, sometimes it's also called the worker node as its where the actual work happens. Each node has multiple Pods on it. There are 3 processes that must be installed in each worker Node that are used to manage and schedule Pods:
- 1=container-runtime e.g. [docker runtime](https://github.com/paguerre3/dockerops)
- 2=KUBLET is a k8s process resposible for scheduling Pods that interacts with both the container and the machine/node i.e. Kublet takes the configuration and starts the pod with a container inside of it and assigns the resources from the machine/node to the container like CPU, RAM and storage resources

**NOTE**
> communication in k8s cluster is made via Services, i.e. that services are responsible for routing and balancing the communication among pods of containerized applications. In other words, services act like load balancers that catch the request directly from a pod and then forward it to another pod, e.g. response communication routed between a pod of a containerized mongodb and another pod of a containerized java application
- 3=KUBE-PROXY is a k8s cluster process responsible for forwarding requests from Services to Pods that must also be installed inside each worker node. Kube-proxy has intelligence inside of it for ensuring that the communication is performant and with low overhead, e.g. if pod of an application intends to communicate with another pod of a data base in k8s cluster, Kube-proxy will forward the communication to a pod of a data base that belongs to the same worker node within the cluster instead of selecting randomly the worker node in the entire cluster
<img src="https://github.com/paguerre3/kubeops/blob/main/support/8-k8s-cluster-processes.PNG" width="43%" height="30%">

- MASTER-NODE or a master machine/server has 4 master-processes installed that control k8s cluster state and the worker nodes:
- 1=API-SERVER is the cluster-gateway that clients use to interact with k8s cluster for deploying an application or querying it, e.g. using a ui like kubernetes dashboard or a command line tool like kublet or a kubernetes api. In other words, the Api Server gets the initial request from a client, e.g. update or query, and then it interacts with k8s custer. It aso acts like a gatekeeper for authentication, i.e. that every request to k8s cluster is being validated by the Api Server of the Master Node and only if validation passes then the communication to other processes is allowed, e.g. request for scheduling a pod or querying cluster state. It is considered good for security because there is only one entry point into the cluster
- 2=SCHEDULER has the intelligence for deciding where to put the pods, i.e. under which worker nodes the pods will be assigned, e.g. it will check CPU, RAM, and storage resources usage and then it will put new pods under worker nodes that are less bussy and have more resources availability. Scheduler "just decides" on which worker Node the Pod should be scheduled and then Kublet is the process on the worker node that actually does the schedule i.e Kublet is resposible for starting the pod
3=CONTROLLER-MANAGER detects cluster state chnages, e.g. when a pod crashes for re-scheduling as soon as possible. Controller Manager communicates with Scheduler for informing the need of scheduling a new pod after a pod crashes on a worker node, then Scheduler decides based on availability under which worker node the schedule should occur and, finally, Kublet will do the schedule on the worker node selected
- 4=ETCD is a key/value store of the cluster state. It is considered the "cluster brain" because each time k8s cluster state changes the information is saved under ETCD, i.e. that cluster changes get stored in the key/value store. All mechanisms of schedulling and cluster management work because its data that is being saved in ETCD, e.g. how does Scheduler know what resources are available? how does Controller Manager know when cluster state changes? or how does Api Server know if cluster is healthy? all the information of k8s cluster state for responding the mentioned questions are stored in ETCD

**NOTE**  
> application data isn't stored in ETCD!
<img src="https://github.com/paguerre3/kubeops/blob/main/support/9-master-processes.PNG" width="48%" height="30%">

- Master Nodes are crucial for k8s cluster operation so, in practice, the cluster is made of multiple Master Nodes where Api Servers are load balanced and ETCD(s) is a distributed storage accross all Master Nodes. Normally Master Nodes require less resources, e.g. CPU or RAM, than worker Nodes but they are considered more important. A k8s cluster small setup usually includes 2 master nodes and 3 worker nodes and while the project increases more Master and worker Nodes are being added maintaining the relationship      