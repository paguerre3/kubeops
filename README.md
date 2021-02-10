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
<img src="https://github.com/paguerre3/kubeops/blob/main/support/12-status-change-of-replicas.PNG" width="48%" height="30%">  

**NOTE**
> attributes of the <code>spec</code> are specific to a Kind of the component! e.g. Deployment kind has its own spec attributes that differ from a Service kind. Status information for re-scheduling and configuration management come from ETCD, i.e. ETCD master process holds the current status of any k8s component! yamel configuration files are usually placed with the code, i.e. IaC=infrastructure as a code. Depending on the project sometimes these are placed within a separated git repository
- <code>template</code> section of a Deployment has its own <code>metadata</code> and <code>spec</code> sections, i.e. a configuration inside another configuration file. The reason of a <code>template</code> section as its own configuration is because it applies to a Pod, i.e. it represents the "blueprint" of a Pod where the image:version, port to open and the name of the container are defined under the <code>containers</code> section of it
- <code>labels</code> and <code>selector</code> are the "connecting" components, e.g. connecting Deployment to Pods or Deployment to a Service. In <code>metadata</code>:<code>labels</code> its defined any key-value pair for components, e.g. <code>app: nginx</code>. A-template-connector) Pods get the <code>label</code> through the <code>template</code> blueprint and "this" <code>label</code> is matched by the <code>selector</code>, e.g <code>selector</code>:<code>matchLabels</code>:<code>app: nginx</code>. B-service-connector) The <code>metadata</code>:<code>labels</code> defined in top section of the Deployment matches with the a Service configuration file <code>spec</code>:<code>selector</code>, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/13-connecting-deployment-service.PNG" width="68%" height="60%">

- Service=<code>ports</code>:<code>port</code> is the Port that the service uses to receive requests from services of other pods and the Service=<code>ports</code>:<code>targetPort</code> is the Port used to forward to the container of the Pod that received the request from the service, i.e. is actually the Port where the container is running therefore it matches with Deployment=<code>spec</code>:<code>template</code>:<code>spec</code>:<code>ports</code>:<code>containerPort</code>, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/14-ports-in-service-and-pods.png" width="73%" height="70%">

**NOTE**
> in case of Service set <code>apiVersion: v1</code> instead of <code>apiVersion: apps/v1</code> in order to avoid unrecognized Kind error
- e.g. [nginx Deployment](https://github.com/paguerre3/kubeops/blob/main/nginx-deployment.yml) + [nginx Service](https://github.com/paguerre3/kubeops/blob/main/nginx-service.yml) execution <code>kubectl apply -f .\nginx-deployment.yml</code> + <code>kubectl apply -f .\nginx-service.yml</code>
- to check services <pre><code>kubectl get service
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   26h
nginx-service   ClusterIP   10.100.89.39   <none>        80/TCP    13s</code></pre>
- to check if service forwards to the right port <pre><code>kubectl describe service nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                10.100.89.39
Port:              <unset>  80/TCP --> service port
TargetPort:        8080/TCP --> target port matches with container ports of Pods
Endpoints:         172.17.0.6:8080,172.17.0.7:8080 --> replica Pods
Session Affinity:  None
Events:            <none></code></pre>
- to check pod with "more details" including IPs<pre><code>kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-f4b7bbcbc-8rch5   1/1     Running   0          20m   172.17.0.6   minikube   <none>           <none>
nginx-deployment-f4b7bbcbc-j6cln   1/1     Running   0          20m   172.17.0.7   minikube   <none>           <none></code></pre>

**NOTE**
> o=output
- to check status updated of the deployment "from ETD"<pre><code>kubectl get deployment nginx-deployment -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
...
labels:
&nbsp;app: nginx
...
status:
&nbsp;availableReplicas: 2
&nbsp;conditions:
&nbsp;\- lastTransitionTime: "2021-02-08T23:15:02Z"
&nbsp;&nbsp;lastUpdateTime: "2021-02-08T23:15:02Z"
&nbsp;&nbsp;message: Deployment has minimum availability.
&nbsp;&nbsp;reason: MinimumReplicasAvailable
&nbsp;&nbsp;status: "True"
&nbsp;&nbsp;type: Available
&nbsp;\- lastTransitionTime: "2021-02-07T23:46:07Z"
&nbsp;&nbsp;lastUpdateTime: "2021-02-08T23:15:03Z"
&nbsp;&nbsp;message: ReplicaSet "nginx-deployment-f4b7bbcbc" has successfully progressed.
&nbsp;&nbsp;reason: NewReplicaSetAvailable
&nbsp;&nbsp;status: "True"
&nbsp;&nbsp;type: Progressing
&nbsp;observedGeneration: 4
&nbsp;readyReplicas: 2
&nbsp;replicas: 2
&nbsp;updatedReplicas: 2</code></pre>

**NOTE**
> to save deployment status from ETCD into a file <code>kubectl get deployment nginx-deployment -o yaml > [fileName]</code>. Warning, if u want to use a deployment based on the outputFile from k8s/ETCD its required to remove all data auto-genereted by k8s which isn't something suggested to do in practice as there are a lot of sections added automatically as status and timestamp
- for deleting components<code>kubectl delete -f [k8s-config-file-name]</code>, e.g. <code>kubectl delete -f .\nginx-service.yml</code> and <code>kubectl delete -f .\nginx-deployment.yml</code>


---
# demo project, i.e. mongodb and mongoexpress
<img src="https://github.com/paguerre3/kubeops/blob/main/support/15-demo-project-flow.png" width="48%" height="30%">

**NOTE**
> <code>kubectl get all</code> returns all components in k8s  
- 1=create Secret for user and password of mongodb so credentials aren't set in plain text under Deployment, i.e. Secret must be created before the Deployment if referenced inside of it (order of creation matters!). A Secret is usually stored in a secured pace different than the Depoyment respository. Warning, storing the data in Secret component doesn't automatically make it secure, there are built-in mechanism (like encryption) for basic security that aren't set by default. Values stored in key/value section of Secret should be written in bas64 encoding format, e.g. <code>echo -n 'username' | base64</code> and <code>echo -n 'password' | base64</code>. e.g. [mongo Secret](https://github.com/paguerre3/kubeops/blob/main/mongo-secret.yml) + execution <pre><code>kubectl apply -f .\mongo-secret.yml
secret/mongodb-secret created</code></pre>

**NOTE**
> <code>kubectl get secret</code> shows secret created
- 2=create Deployment yaml, e.g. [mongo Deployment](https://github.com/paguerre3/kubeops/blob/main/mongo-deployment.yml) + execution <pre><code>kubectl apply -f .\mongo-deployment.yml
deployment.apps/mongodb-deployment created</code></pre>

**NOTE**
> <code>kubectl get pod --watch</code> to observe progress of pod creation
- 3=create InternalService so other commponents can talk "internally" to mongodb, like mongoexpress. Recommendation is to place InternalService into the same yamel of the Deployment as they usually belong together, i.e. rename <code>mongo-deployment-yml</code> to <code>mongo.yml</code>, e.g. [mongo Deployment + InternalService](https://github.com/paguerre3/kubeops/blob/main/mongo.yml) and execution <pre><code> kubectl apply -f .\mongo.yml
deployment.apps/mongodb-deployment unchanged --> deployment haven't been changed, only Service was added 
service/mongodb-service created</code></pre>

**NOTE** 
> <code>kubectl get all | grep mongodb</code> to filter output results by "mongodb" in unix like OOSS. <code>kubectl get all -o wide</code> to show more details including IPs
- 4=create ConfigMap for storing mongodb address to be used by "mongoexpress" Deployment for communicating with mongodb. Note that ConfigMap is used as a centralized store for common configurations among Deployments/Pods because the Deployment itself can hold the URL value directly but its a less preferred option than ConfigMap as if other Deployments/Pods need to use the same address it won't be shared. Warning, if ConfigMap reference is used inside a Deployment then ConfigMap must be created in 1st place! e.g. [mongo ConfigMap](https://github.com/paguerre3/kubeops/blob/main/mongo-configmap.yml) and execution <pre><code>kubectl apply -f .\mongo-configmap.yml
configmap/mongodb-configmap created</code></pre>
- 5=create "mongoexpress" Deployment for communicating with mongodb, using ConfigMap as reference, and also create "External"Service inside same yamel so mongoexpress ui can be accessed from browser, e.g. [mongoexpress Deployment + ExternalService](https://github.com/paguerre3/kubeops/blob/main/mongoexpress.yml) and execution <pre><code>kubectl apply -f .\mongoexpress.yml
deployment.apps/mongoexpress-deployment created
service/mongoexpress-service created</code></pre>
- Service=<code>spec</code>:<code>type: LoadBalancer</code> makes the "External"Service. This is considered a bad name for <code>type</code> because InternalService also loads and balances requests. <code>LoadBalancer</code> accepts external requests by assigning an "external" IP to the Service. Important, its also necessary to add a <code>nodePort</code> inside Service=<code>spec</code>:<code>ports</code> section that refers to the "external" port to be accessed from browser but it only has a "valid range" of assignment, i.e. 30000-32767!
- <code>ClusterIP</code> is the default type for InternalService that provides an "internal" service IP while <code>LoadBalancer</code> provides "external" IP and also internal IP (both) for the ExternalService, e.g. <pre><code>kubectl get service
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes             ClusterIP      10.96.0.1        \<none\>        443/TCP          47h
mongodb-service        ClusterIP      10.97.179.81     \<none\>        27017/TCP        6h10m
mongoexpress-service   LoadBalancer   10.108.230.111   \<pending\>     8081:30000/TCP   16m --> it only shows "pending" EXTERNAL IP because of the usage of Minikube (using k8s directly should display external IP right away)</code></pre>

**NOTE**
> 6="only for Minikube" (because it shows "pending") is needed to additionally execute "manually" <code>minikube service mongoexpress-service</code> so Minikube assigns the external IP to the ExternalService of mongoexpress already defined, e.g. using docker driver and tunneling
<img src="https://github.com/paguerre3/kubeops/blob/main/support/16-minikube-assign-external-ip.PNG" width="48%" height="30%">


---
# k8s namespaces
- resources can be organized by namespaces, e.g. services from different business units or kind of resources 
- namespace is a virtual cluster inside of a k8s cluster
- check namespaces<pre><code>kubectl get namespace
NAME              STATUS   AGE
default           Active   2d3h
kube-node-lease   Active   2d3h
kube-public       Active   2d3h
kube-system       Active   2d3h</code></pre>
- kube-system=a k8s user should never create or update anything from it as it holds System processes, i.e. Master and Kubectl processes
- kube-public=it contains public accessible data. It has a ConfigMap that contains cluster information that can be accessed even without authentication, e.g. <pre><code>kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:49153
KubeDNS is running at https://127.0.0.1:49153/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy</code></pre> 
- kube-node-lease=it holds information about heart-beats of Nodes. Each Node has associated lease/contact object (heart-beat) in namespace that informs Availability information of it
- default=resources created without defining any namespace are placed under this location
- namespace creation command <code>kubectl create namespace [namespace]</code>
- another way of creating namespaces is using a configuration file, e.g. [my\-namespace](https://github.com/paguerre3/kubeops/blob/main/namespace.yml) and execution <pre><code>kubectl apply -f .\namespace.yml
namespace/my-namespace created</code></pre> 
... and then do the reference in related components under <code>metadata</code>:<code>namespace</code> section
<img src="https://github.com/paguerre3/kubeops/blob/main/support/17-namespace-in-configfile.PNG" width="28%" height="30%">

- Use cases: 1=Structure/Grouping by kind of resource, e.g. database, monitoring and nginx, 2=Avoid conflicts of many teams using the same application, i.e. group components by teams, e.g. ProjectA and ProjectB so configuration files aren't overwritten during deployment, 3.A=Resource sharing, i.e. sharing a group of resources into different environments, e.g. sharing nginx or monitoring stack namespaces into Development and Staging environment as those are common resources that sometimes can be shared, 3.B=Resource sharing in case of Blue/Green deployment, i.e. having in production "current" and "next" release clusters (both running at the same time) sharing common resources as nginx and monitoring, 4=access and resource limits, i.e. resources limited by teams so they have their own secured/isolated environment and therefore they can't delete configurations of other projects, e.g. ProjectA can't create nor delete configurations of ProjectB, and additionally, each project can set its own limits of CPU, RAM and storage per NS so their costs can be monitored separately, e.g. ResourceQuota-ProjectA and ResourceQuota-ProjectB   

**NOTE**
> Small projects with less than 10 users don't require the use of namespaces
- Namespace characteristics: a namespace "can't access most resources" from another namespace, e.g. ConfigMap/Secret for Services of one namespace that access a Database can't be reused in another namespace even if they point to the same Component, i.e. important, each namespace must define its own ConfigMap or Secret!
<img src="https://github.com/paguerre3/kubeops/blob/main/support/18-each-namespace-has-own-configmap.PNG" width="48%" height="30%">

- Services are components "allowed" to be shared among resources of other namespaces, e.g. Services can be referenced from a ConfigMap of another NS by specifying the namespace "prefix" in the value section of the ConfigMap=<code>data</code>:<code>custom-key</code> section
<img src="https://github.com/paguerre3/kubeops/blob/main/support/19-service-shared-among-namespaces.PNG" width="73%" height="70%">

- Some components can't be created within a namespace because they live globally in k8s cluster, e.g. Volume/persistent-volume and Node resources. To check the complete list of components that can't be bouded to an specific namespace <code>kubectl api-resources --namespaced=false</code> and to check the resources that are admitted to be created by namespace its used the "same command" but with flag enabled <code>--namespaced=true</code>

**NOTE**
> for the use case of having one team assigned to an specific namespace entirely it might be useful to change the "active" default namespace so there is no need of specifying it in command line, e.g. for avoiding doing <code>kubectl get pod -n [namespace]</code>. The tool out of the box named "kubens" is used for switching the active NS while "kubectx" is used for swithing cluster contexts (once kubectx is installed it comes with kubens), e.g. <code>kubens [namespace]</code> switches to [namespace] as active


---
# k8s ingress        