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

- Master Nodes are crucial for k8s cluster operation so, in practice, the cluster is made of multiple Master Nodes where Api Servers are load balanced and ETCD(s) is a distributed storage accross all Master Nodes. Normally Master Nodes require less resources, e.g. CPU or RAM, than worker Nodes but they are considered more important. A k8s cluster small setup usually includes 2 master nodes (at least) and 3 worker nodes and while the project increases more Master and worker Nodes are being added maintaining the relationship


---
# Minikube and kubectl
- Minikube is a test/local setup cluster where master + worker processes live only in "one" Node/Machine so there is no need to build a "real" k8s cluster with multiple Master and worker Nodes that will consume a lot of resources. It also has a docker container runtime pre-installed. In a laptop, minikube runs through Virtual Box or some other Hypervisor, i.e. it creates a virtual box on the laptop and Nodes run in that virtual box. In summary, Minikube is a One Node k8s cluster that runs inside Virtual Box used for Testing kubernetes in a Local Setup that doesn't require a lot of resources
<img src="https://github.com/paguerre3/kubeops/blob/main/support/10-minikube.PNG" width="48%" height="30%">

- KUBECTL is a command line tool used to interact with k8s cluster, e.g. for creating pods, configMap/secret and services. It talks with k8s cluster via Api Server/cluster-gateway. Kubectl is also considered the most powerful client of Api Server compared to kubernetes dashboard ui and also the apis

**NOTE**
> Kubectl is used for interacting with a "real" k8s cloud cluster and, also, for speaking with Minikube in case of a test/local setup. Minikube install includes kubectl as a dependency
- [Minikube install](https://minikube.sigs.k8s.io/docs/start/)
- once installed, virtualization mechanism can be provided during minikube initialization, e.g. using docker driver for virtualization<pre><code>minikube start --vm-driver=docker</code></pre>
- command for getting nodes status is<pre><code>kubectl get nodes
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   12m   v1.20.2</code></pre>
- check global status of processes<pre><code>minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
timeToStop: Nonexistent</code></pre>
- check version<pre><code>kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:20:00Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}</code></pre>


---
# Main kubectl commands
- check nodes staus <code>kubectl get nodes</code>
- check pods<pre><code>kubectl get pod
No resources found in default namespace</code></pre>
- check services<pre><code>get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   26m</code></pre>
- to create components is used <code>kubectl create</code> but Pod component isn't listed on --help as it can only be created using Deployments in case of stateLESS containerized applications or StatefulSet in case of sateFUL applications, e.g. usage <code>kubectl create deployment NAME --image=image [--dry-run] [options]</code>

**NOTE**
> <code>--dry-run</code> doesn't really run the process and instead it prints it like if it was running just to check/test if its ok for running without persisting anything
- e.g. create Deployment/node based on nginx:latest dockerhub image<pre><code>kubectl create deployment nginx-depl --image=nginx
deployment.apps/nginx-depl created</code></pre>
- check deployments<pre><code>kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           83s</code></pre>
- check pods now shows one with prefix "ngnix-depl"<pre><code>kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-5c8bf76b5b-45vrs   1/1     Running   0          2m47s</code></pre>

**NOTE**
> a valid pod status is ContainerCreating in case it wasn't Running yet
- between Deployments and pods there is another abstraction layer called ReplicaSet, i.e. <pre><code>kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   1         1         1       8m29s</code></pre>

**NOTE**
> Pod NAME is made of ReplicaSet name+its own id
- REPLICASET manages the replicas of a Pod. In practice, k8s users never create ReplicaSet(s) and instead they work directly with Deployment definitions, i.e. a Deployment is the blueprint of the Pod and its configuration includes the specification about how many replicas are needed <code>[options]</code>
- Layers of abstractions (top-to-bottom): a Deployment manages a ReplycaSet, a ReplicaSet manages the number of Pod replicas, and, a Pod manages a containerized appliction, i.e. that everything bellow Deployment is handled ny kubernetes! e.g. edit works at Deployment level, i.e. doing <code>kubectl edit deployment nginx-depl</code> and then changing the image number to an specific version+save will automatically trigger an update of the current pod that holds the different containerized image version/requesting a new download and then replacing current running pod with another version with the same Deployment "prefix" but with a new SeplicaSet number + new Pod id as a suffix <pre><code>kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   0         0         0       40m --> NO pod exists in the old replicaset
nginx-depl-7fc44fc5d4   1         1         1       5m22s</code></code>
- debugging command is <code>kubectl logs [pod-name]</code>, e.g. 1st, create another deployment because nginx doesn't store logs by default, i.e. <code>kubectl create deployment mongo-depl --image=mongo</code> and then for checking logs<pre><code>kubectl logs mongo-depl-5fd6b7d4b4-cmsv8
{"t":{"$date":"2021-02-07T22:33:34.095+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
...</code></pre>
- <code>kubectl describe pod [pode-name]</code> provides further information regarding pod status changes that could be useful when pod is in ContainerCreating state, e.g. <pre><code> kubectl describe pod mongo-depl-5fd6b7d4b4-cmsv8
Name:         mongo-depl-5fd6b7d4b4-cmsv8
...
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  8m24s  default-scheduler  Successfully assigned default/mongo-depl-5fd6b7d4b4-cmsv8 to minikube
  Normal  Pulling    8m24s  kubelet            Pulling image "mongo"
  Normal  Pulled     7m34s  kubelet            Successfully pulled image "mongo" in 50.4935322s
  Normal  Created    7m34s  kubelet            Created container mongo
  Normal  Started    7m33s  kubelet            Started container mongo</code></pre>
- <code>kubectl exec -it [pode-name] -- bin/bash</code> is used to access inside of the containerized application of a pod by name, e.g. <pre><code>kubectl exec -it mongo-depl-5fd6b7d4b4-cmsv8 -- bin/bash
root@mongo-depl-5fd6b7d4b4-cmsv8:/#</code></pre> 

**NOTE**
> it=interactive terminal. Once inside do <code>env</code> to get all environment variables inside the containerized application and <code>exit</code> to leave the terminal
- <code>kuebctl delete deployment [deployment-name]</code> for removing the pod and all it's abstraction layers, i.e. removal include deployment, replicaset and pod, e.g. <pre><code>kubectl delete deployment nginx-depl
deployment.apps "nginx-depl" deleted</code></pre>

**NOTE**
> CRUD operations happen at deployment level, everything underneath is managed by k8s!
- <code>kubectl apply -f [k8s-config-file.yml]</code> is the command used "in practice" to create Deployment, StatefulSet, Services, etc, considered simpler to use than providing all [options] in command line. It actually executes whatever is specified in the k8s-config-file.yml, e.g. [nginx Deployment](https://github.com/paguerre3/kubeops/blob/main/nginx-deployment.yml) + fulfilment <pre><code>kubectl apply -f nginx-deployment.yml
deployment.apps/nginx-deployment created</code></pre>

**NOTE**
> f=file
- then, based on previous example, if its required to update the number of replicas this can simply be done by changing it in the Deployment file and doing <code>apply</code>, i.e. k8s knows when to create or update deployments, e.g. <pre><code>kubectl apply -f nginx-deployment.yml
deployment.apps/nginx-deployment configured --> now it doesn't say created and instead this means update</code></pre> and checking pods results <pre><code>kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
mongo-depl-5fd6b7d4b4-cmsv8         1/1     Running   0          87m
nginx-deployment-644599b9c9-95vxt   1/1     Running   0          2m24s 
nginx-deployment-644599b9c9-w8jzm   1/1     Running   0          14m --> same replicaset id=nginx-deployment-644599b9c9 for both pods as they are replicated</code></code>
- summary of main k8s commands
<img src="https://github.com/paguerre3/kubeops/blob/main/support/11-kubectl-commands-1.PNG" width="48%" height="30%">
<img src="https://github.com/paguerre3/kubeops/blob/main/support/11-kubectl-commands-2.PNG" width="48%" height="30%">


---
# Yamel configuration file in k8s
- main tool for configuring components in k8s cluster, it has 3 parts: 
- 1=<code>metadata</code> is where the name of the component is defined
- 2=<code>spec</code> or specification is where the configurations of the component to apply exist, e.g. number of replicas. 
- 3=<code>status</code> is automatically generated and added by kubernetes, i.e. k8s compares what is the "desired" state vs the "actual" state of a file so if they don't match then k8s knows that there is something that has to be fixed/updated (considered the basis of self-healing features of k8s), e.g. when incrementing the number of replicas of a Deployment so k8s knows that a new replica has to be added:
<img src="https://github.com/paguerre3/kubeops/blob/main/support/12-status-change-of-replicas.PNG" width="43%" height="30%">  

**NOTE**
> attributes of the <code>spec</code> are specific to a Kind of the component! e.g. Deployment kind has its own spec attributes that differ from a Service kind. 
> Status information for re-scheduling and configuration management come from ETCD.  

