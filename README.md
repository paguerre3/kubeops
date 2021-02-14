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
> for the use case of having one team assigned to an specific namespace entirely it might be useful to change the "active" default namespace so there is no need of specifying it in command line, e.g. for avoiding doing <code>kubectl get pod -n [namespace]</code>. The tool out of the box named "kubens" is used for switching the active NS while "kubectx" is used for switching cluster contexts (once kubectx is installed it comes with kubens), e.g. <code>kubens [namespace]</code> switches to [namespace] as active


---
# k8s ingress
- Ingress is used in real production environments where having ExternalService for expossing "external" IP isn't adequate, i.e. normally an application is accessed setting its domain name and secured port through the client browser, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/20-ingress.PNG" width="48%" height="30%">

**NOTE**
> the request received by the Ingress is then forwarded to an InternalServices that will finally communicate with the Pod
- Ingress configuration if of <code>kind: Ingress</code>, and then <code>spec</code> contains a section <code>rules</code> where "routing rules" have defined <code>-host</code> domain addresses that receive requests and forwards to <code>http</code>:<code>paths</code>:<code>-backend</code>:<code>serviceName</code> and <code>servicePort</code>, i.e. Routing rules forward requests to InternalService/s. In other words in Ingress its defined a mapping that forwards requests from Host to InternalService. <code>paths</code> section also holds anything that comes after the domain name, e.g. path resources. Warning, Ingress=<code>spec</code>:<code>rules</code>:<code>http</code> doesn't correspond to the "external communication protocol" that public URL uses, e.g. HTTPS or HTTP of my-domain, and instead it belongs to the "internal protocol" being used for forwarding the requests to InternalService. <code>http</code>:<code>paths</code>:<code>-backend</code>:<code>servicePort</code> of Ingress matches with InternalService <code>port</code> and not <code>targetPort/node</code> nor <code>nodePort/exernalIP</code> (when <code>LoadBalancer</code> type of ExternalService). e.g. [sample of Ingress](https://github.com/paguerre3/kubeops/blob/main/ingress.yml)
<img src="https://github.com/paguerre3/kubeops/blob/main/support/21-ingress-internal-service-mapping.PNG" width="73%" height="70%">

**NOTE**
> <code>host</code> present in <code>routes</code> of Ingress should be a valid domain address. Important, <code>host</code> maps domain name to a Node's IP address which is considered the "entry point" OR, alternatively, <code>host</code> maps domain name to a server outside of k8s cluster that acts like a Proxy or Secured Gateway that behaves as "entry point", i.e. Ingress will receive request from the internal or external "entry point"/host and then it will forward to InternalService
- In order to work Ingress needs an "implementation" of it which is an IngressController Pod or set of Pods, i.e. IngressController runs on Pod or a Set of Pods of a Node in k8s cluster and does the "evaluation and processing" of Ingress rules. In other words IngressController is the "entry point" of k8s cluster that evaluates all rules and manages redirections. There are many third-party implementations of IngressControllers and k8s also offers its own implementation which is "Nginx" IngressController therefore it needs to be installed so Ingress can function
<img src="https://github.com/paguerre3/kubeops/blob/main/support/22-ingress-controller.PNG" width="48%" height="30%">

- is important to understand the environment on which the cluster runs, i.e. if k8s cluster runs under a Cloud Service Provider like AWS or Google Cloud that have out-of-the-box kubernetes solutions or that use their own virtualized load balancer then there is normally a Cloud Provider "Load Balancer" placed in front of k8s cluster that behaves as a Secured Load Balancer "entry point" that receives and forwards requests to the IngresController of k8s, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/23-ingress-controller-cloud-provider.PNG" width="48%" height="30%">

**NOTE**
> The advantage of using a Cloud Provider is that the user doesn't need to create a "custom" Load Balancer so the setup is simplified. Doing a "bare metal" deployment requires some kind of "entry point" configuration that, as mentioned, can be placed "outside" or "inside" the cluster, e.g. a Proxy Server entry-point outside k8s cluster or a Node IP entry-point
<img src="https://github.com/paguerre3/kubeops/blob/main/support/24-ingress-controller-bare-metal-proxy.PNG" width="48%" height="30%">

- 1=install IngressController in Minikube so Ingress can work, i.e. it automatically starts the Nginx implementation of IngressController<pre><code>minikube addons enable ingress
\* After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
\* Verifying ingress addon...
\* The 'ingress' addon is enabled</code></pre>
- "at the end of all steps" enable tunneling if needed for testing purposes<code>minikube tunnel</code>
- check Nginx IngressController is running under "kube-system" NS<pre><code>kubectl get pod -n kube-system
NAME                                        READY   STATUS      RESTARTS   AGE
coredns-74ff55c5b-67w9j                     1/1     Running     6          3d4h
etcd-minikube                               1/1     Running     6          3d4h
ingress-nginx-admission-create-dcxqv        0/1     Completed   0          12m
ingress-nginx-admission-patch-rk9kq         0/1     Completed   0          12m
ingress-nginx-controller-558664778f-dr57v   1/1     Running     0          12m
kube-apiserver-minikube                     1/1     Running     6          3d4h
kube-controller-manager-minikube            1/1     Running     6          3d4h
kube-proxy-xqrnr                            1/1     Running     6          3d4h
kube-scheduler-minikube                     1/1     Running     6          3d4h
storage-provisioner                         1/1     Running     12         3d4h</code></pre>
- 2=enable k8s dashboard and metrics-server (dependency of dashboard) in Minikube to do a Demo of Ingress configuration, i.e. execute <code>minikube addons enable dashboard</code> and then <code>minikube addons enable metrics-server</code>. To check the list of Minikube enabled addons <code>minikube addons list</code>

**NOTE**
> k8s dashboard has InternalService and Pod already configured but it doesn't have a Ingress/IngressController enabled
- check NS in order to visualize the one associated to the dashboard<pre><code>kubectl get namespace
NAME                   STATUS   AGE
default                Active   3d5h
kube-node-lease        Active   3d5h
kube-public            Active   3d5h
kube-system            Active   3d5h
kubernetes-dashboard   Active   23m
my-namespace           Active   25h</code></pre>
- check configurations filtered by dashboard NS (Ingress "rule" configuration isn't present)<pre><code>kubectl get all -n kubernetes-dashboard
NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-c95fcf479-kk692   1/1     Running   0          25m
pod/kubernetes-dashboard-6cff4c7c4f-9vpvg       1/1     Running   0          25m
NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.100.171.241   none          8000/TCP   25m
service/kubernetes-dashboard        ClusterIP   10.99.44.121     none          80/TCP     25m
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           25m
deployment.apps/kubernetes-dashboard        1/1     1            1           25m
NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-c95fcf479   1         1         1       25m
replicaset.apps/kubernetes-dashboard-6cff4c7c4f       1         1         1       25m</code></pre>
- 3=create Ingress "rule" resource for k8s dashboard, i.e. [dashboard Ingress](https://github.com/paguerre3/kubeops/blob/main/dashboard-ingress.yml) and execute <pre><code>kubectl apply -f .\dashboard-ingress.yml
Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/dashboard-ingress created</code></pre>
- check Ingress rule creation<pre><code> kubectl get ingress -n kubernetes-dashboard
NAME                CLASS    HOSTS           ADDRESS        PORTS   AGE
dashboard-ingress   none     dashboard.com   192.168.49.2   80      3m7s</code></pre>
- 4=emulate "entry point" that behaves as a Proxy in front of IngressController outside k8s cluster so IngressController can use "dashboard" Ingress rule to evaluate and manage redirection (forwarding requests to "dashboard" InternalService), i.e. go to "hosts" file of os and create dns rule that matches with HOST and IP address of dashboard-ingress, e.g. 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/25-hosts-as-proxy.PNG" width="43%" height="40%">

- 5=open browser, write domain "dashboard.com" and check k8s dashboard. 

**NOTE**
> [hello example of Ingress "v1" latest-version](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/). Additionally, using "v1" latest api version of Ingress, in section Ingress=<code>spec</code> can be defined a <code>defaultBackend:</code> where a Service can be referenced/pointing to a Pod that responds with error codes in case of attempts to access resources inside the host that don't exist, i.e. it behaves as a "default-http-backend" Service that responds with meaningful error responses
- Ingress use cases offer: A. support of multiple paths for same host, i.e different resources/paths that map with different services/pods
<img src="https://github.com/paguerre3/kubeops/blob/main/support/26-ingress-same-host-multiple-paths.PNG" width="48%" height="30%">

-B. multiple sub-domains or domains, i.e. each host represents a subdomain
<img src="https://github.com/paguerre3/kubeops/blob/main/support/27-multiple-domains.PNG" width="48%" height="30%">

-B. configuring TLS certificate, i.e. configuring HTTPS protocol forwarding in Ingress, e.g. 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/28-ingress-tls-certificate.PNG" width="73%" height="70%">

**NOTE**
> Secret stores certificate <code>tls.crt</code> and <code>tls.key</code> encoded in base64 "in situ", i.e. complete data is placed encoded in yamel instead of pointing to a reference location. Keys defined must be exactly like that (<code>tls.crt</code> and <code>tls.key</code>), not like most other secret mappings where the keys could be defined by the k8s user and, finally, Secret=<code>type: kubernetes.io/tls</code> (must be of kind "tls"). Warning, Secret must be placed in the same Namespace where the Ingress component is defined!


---
# helm package manager
- Helm is a package manager (e.g. apt or yum) for k8s yamels, i.e. is a convenient manner for Packaging yamel files and distribute them in public and private repositories, e.g. Elastick Stack for Logging (including "all yamel files"=StatefulSet, ConfigMap, k8s user with permissions, Secret and Services related to the same Elastick namespace) is packaged in a repository as its considered a common solution/standard that can be reused in different k8s clusters 
- The bundle of yamel files (package) is called Helm Charts, i.e. using Helm, developers create their own Helm Charts (bundle of yamel files), push them into Helm Repositories, and download them so Helm Charts can be reused easily by others, e.g. database applications as mongodb, elasticksearch and mysql, or monitoring applications like prometheus, they are all available as Helm Charts in Helm Repositories
- check Helm Charts in [helm hub](https://helm.sh/docs/helm/helm_search_hub/)
- Helm is also considered a Templating Engine as all yamels of Deployment and Service configuration files (e.g. for building Pods of microservices) are pretty similar in their Structure (they mainly differ in docker Image reference secion for selecting Image:version-tag to download and their label names) therefore without Helm k8s users need to write each Deployment.yml separatelly (including its Service definition usually inside Deployment), i.e. using Helm helps developers so they define a common "blue print" (template yamel) and then only "dynamic values" are replaced by Placeholders so there is no need of writting so many yamel files, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/29-helm-template.PNG" width="73%" height="70%">   

<img src="https://github.com/paguerre3/kubeops/blob/main/support/30-helm-template-values.PNG" width="73%" height="70%">

**NOTE**
> Helm is useful within CI / CD pipelines because in the builds the values of the Template.yml file can be replaced "on the fly" by the values.yml Placeholders
- Another use case is using the same application across different environments (k8s clusters), e.g. k8s cluster of Development, Staging and Production could share the same Helm Chart (deployment bundle) so the "same" package can be deployed/re-used under different environments executing just one command
- Helm structure has a <code>top level folder</code> with the name of the chart, <code>Chart.yml</code> with meta-info of chart, <code>values.yml</code> containing default values to be replaced in the template, <code>charts</code> folder with dependencies from other charts, and <code>templates</code> folder that has all the template files of the chart, i.e.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/31-helm-chart-structure.PNG" width="73%" height="70%">

- command to install Chart into k8s <code>helm install [chart-name]</code>. It take template.yml files and fills them with values.yml, i.e. producing valid k8s manifests (blueprints) ready to be deployed in k8s. Optionally, Charts might include readme and license
- default values.yml can be overriden during install command execution or using a command flag which is a less preferred option, e.g. <code>helm install -values=my-values.yml [chart-name]</code> will overritte only the values present in my-values.yml making a merge with values.yml default, i.e.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/32-helm-values-override.PNG" width="73%" height="70%">

**NOTE**
> another feature of Helm is release management only for Helm 2 by deploying Tiller Server inside k8s cluster, which it was responsible for handling Charts release versions (providing stupport to upgrade and rollback of bundles) but Tiller was depcrecated in Helm 3 because of security concerns (too much power inside k8s, i.e. Tiller could create, update and delete components having too much permissions)


---
# k8s volumes  
- Storage doesn't depend on the Pod life cycle (because Pods are ephemeral)
- Storage must be available on all nodes
- Storage needs to survive even if k8s cluster crashes
- Persistance storage also applies for writting/reading directories, e.g. a folder path
- Persistent Volume is a k8s cluster resource, like RAM or CPU, used to "store" data
- PersistentVolume gets created from a yamel file, i.e. <code>kind: PersistentVolume</code> and its <code>spec</code> defines how much storage to "use", e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/33-persistent-volume.PNG" width="48%" height="30%">

- PersistentVolume needs physical storage, e.g. local disk, NFS (Network File System) server for distributed storage or Cloud-storage (aws block storage or google cloud store)
- k8s doesn't care about the technology of the actual storage and its location as it provides PersistentVolume as an interface to access the actual storage, i.e. the administrator of the storage is responsible for configuring it besides using k8s. The administrator will decide the type of storage, and then it will create and manage it, e.g. deciding the backup mechanism and ensuring that it won't be corrupted. In other words, storage in k8s is an "external" plugin into the cluster
- k8s allows multiple Pods/applications to use different type of storages from the same cluster, e.g. 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/34-multiple-storages.PNG" width="48%" height="30%">

- PersistentVolume interface component "uses" the physical storage in the <code>spec</code> section, i.e. it references about "how to use" it, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/35-nfs-storage.PNG" width="73%" height="70%">

- <code>spec</code> attributes differ depending on storage "type", e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/36-gc-storage.PNG" width="73%" height="70%">

<img src="https://github.com/paguerre3/kubeops/blob/main/support/37-local-storage.PNG" width="73%" height="70%">

**NOTE**
> "Node affinity" exists when the local storage refers to a Pod of the same k8s cluster
- Important, Persistent Volumes are not Namespaced, i.e. they are not linked to a particular namespace as they are accessible to the whole k8s cluster
- Warning, Local vs Remote types. Each volume type supports its own "use case" but Local type violates some of the requirements previously explained, i.e. Local storage (it has Node affinity) violates being tied to an specific Node and therefore surviving to cluster crashes. In summary, for data base persistance is recommended to use Remote storage!
- Life cycle: PersistentVolume needs to exist before the Pod that depends on it is created

**NOTE**
> K8s Administrators vs Users: k8s Aministrator sets up and mantain the cluster, i.e. it ensures it has enough resources. K8s Users deploy applications in cluster (directly or using a CI pipeline).
- 1=Storage resources are provisioned by k8s Administrator, e.g. it creates NFS Server or configures a Cloud Storage so they are available for the cluster, and then makes PersistentVolume components for "using" the existent physical Storage backends
- 2=Later on, k8s Users/Developers will configure their applications/Pods for using the PersistentVolume, i.e. applications have to "claim" their PersistentVolume and for doing that k8s offers the PersistentVolumeClaim component. A PersistentVolumeClaim basically "claims" a Volume with an specific storage/usage and whatever PersistentVolume available that "matches the claim/satisfies the criteria" will be used, e.g. 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/38-persistent-volume-claim.PNG" width="73%" height="70%">

- 3=finally, k8s User has to make the reference into the Pod pointing to the PersistentVolumeClaim (PVC), i.e. the Pod needs to have the reference to the PersistentVolumeClaim component, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/39-persistent-volume-claim-pod-reference.PNG" width="73%" height="70%">

**NOTE**
> Warning, a PersistentVolumeClaim (PVC) must exist in the "same namespace" where the Pod that uses it is located!
- levels of persistent volume abstractions
<img src="https://github.com/paguerre3/kubeops/blob/main/support/40-volumes-abstraction-levels.PNG" width="48%" height="30%">

- Once the Claim (PVC "namespaced") is satisfied by a PersistentVolume (PV "global") then a Volume is mounted into the Pod that then is being mounted into the Container, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/41-persistent-volume-mounts.PNG" width="73%" height="70%">

**NOTE**
> In the case of having multiple containers inside the same Pod, is also possible to define if the Volume can be mounted to all the containers inside the same Pod or just to some of them
- ConfigMap and Secret are considered "special cases" of Volumes are they are Local volumes not created via PersistentVolume (PV) and PVC (PersistentVolumeClaim) which are managed by k8s. In a similar sense compared to PV/PVC, they can be mounted as a local volume  if needed, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/42-mount-configmap-secret.PNG" width="73%" height="70%">

- [types of volumes](https://www.tutorialspoint.com/kubernetes/kubernetes_volumes.htm)
- a Pod/application might have mounted "multiple types" of Volumes, e.g. elastic-app
<img src="https://github.com/paguerre3/kubeops/blob/main/support/43-pod-with-multiple-volumes.png" width="73%" height="70%">

- 4=Important, in huge projects, working with a large number of PV and PVC might might be something complex to handle by k8s Administrators and Users, i.e. k8s admins creating and managing a vast amount of physical storage types and their PersistentValues (PVs) while k8s users make their "claims" (PVC) might be chaotic. In order to make this process more efficient there is a 3rd persistent component called "StorageClass" (SC) which provisions "PersistentVolumeS" (PVs) dynamically when PersistentVolumeClaim (PVC) "claims" it, i.e. in this manner, the creation of PersistentVolumeS (PVs) is automatic. Storage backend is defined in the StorageClass (SC) via <code>provisioner</code> attribute. Each storage backend has its own Provisioner, e.g. internal provisioners start with prefix "kubernetes.io" like <code>provisioner: kubernetes.io/aws-ebs</code> while externals have different paths. <code>parameters</code> attribute is the place where StorageClass (SC) configures the parameters for the storage that a k8s user wants to request for PersistentVolume (PV). In other words, a StorageClass is another abstraction level that abstracts underlying storage provider/provisioner as well as <code>parameters</code> for that Storage.


**NOTE**
> aws-ebs=AmazonWS Elastick Blocks Store   
- 5=StorageClass (SC) is requested via PersistentValueClaim (PVC) under <code>spec</code>:<code>storageClassName</code> section, e.g. 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/44-storage-class-and-pvc.PNG" width="83%" height="80%">

- StorageClass usage: 1=Pod claims storage via PVC, 2=PVC requests storage from SC, 3=SC creates "dynamically" the PV that meets the needs of the Claim (instead of waiting for the k8s Administrator to do it manually, i.e. creating PV)
<img src="https://github.com/paguerre3/kubeops/blob/main/support/45-storage-class-usage.PNG" width="73%" height="70%">


---
# k8s statefulSet
- StatefulSet is used in applications that stores data like DataBases, e.g. mongoDb, mySql and elasticsearch. It depends on most up-to-date data/state for responding to query data and updates

**NOTE**
> Stateless applications don't keep record of state as each request received is treated as completely new and sometimes they can forward them to stateful apps. k8s Deployment configuration file is being used for building stateLESS applications as Deployment is an abstraction of Pods for making replications, i.e. Pods are considered ephemeral because they could crash and be replaced so Deployment is being used on stateLESS apps

- k8s StatefulSet configuration is used for deploying stateFUL applications and handlings their replications/managing the run of multiple replicas at the same time
- Deployment and StatefulSet are based on the same Container specification and they both can handle Storage/Volumes the same way so, why is StatefulSet needed for? replicating stateFUL applications is more difficult and it has different requirements than stateLESS applications, i.e. Database Pods can't be created/deletead at the "same" time and they can't be "randomly" addressed/identified because replica Pods aren't identical, they have their own Pod Identity "not-random" (this is the opposite of a Deployment/Pod of stateless applications). In other words, what StatefulSet provides in difference to Deployment is the fact of giving a Pod its own "sticky" Identity, i.e. each Pod has its "sticky" identity which means that even though they are created from same specification they aren't interchangeable. Each StatefulSet pod has its own persistent identifier that remains after crashes, i.e. persistent ID across any re-scheduling
- StatefulSet Pods have "sticky" identity due to Scaling databse applications, i.e. multiple replica Pods can't write a database at the same time because that causes data inconsistency (usually, a master DB acts as writter/reader and slaves are readers)      
<img src="https://github.com/paguerre3/kubeops/blob/main/support/46-statefulset-identity.PNG" width="48%" height="30%">

- Additionally, slave/reader Pods don't use the same physical storage even though they share the same replicated data, i.e. each pod replica must have the same data accross the database cluster at any time having a synchronization mechanism in place, configured by k8s administrator, which normaly maintains data up-to-date after "master" Pods writes.   
<img src="https://github.com/paguerre3/kubeops/blob/main/support/47-statefulset-cluster-sync.PNG" width="73%" height="70%">

**NOTE**
> the creation of a new StatefulSet Pod means clonning the state/data of a previous Worker Pod (slave), i.e. replicating the same data in a new physical storage + new PersistentVolume (PV) + new StatefulSet Worker (slave). Once data is clonned, the synchronization mechanism starts over the new Worker (taking into account any updates of the Master pod). PersistentVolume life cycle isn't tied to others components of k8s cluster so it isn't affected by a Deployment/Pod chrash
- The way a Pod state is maintained after a crash is due to its sticky Identity defined in SatefulSet (SS) which ensures that the PersistentVolume gets re-attached after any re-schedule (together with its physical storage)
<img src="https://github.com/paguerre3/kubeops/blob/main/support/48-statefulset-pv-reattach.PNG" width="48%" height="30%">

**NOTE**
> the re-attachement process is guaranteed via Remote physical storage only as using local storage inside the cluster + pod affinity causes data loss (not recommended by a database use case) 
- StatefulSet (SS) identity is made of "fixed ordered names" while Deployment is a random hash, i.e. <pre><code>${statefulset name}-${ordinal} e.g. mysql-0 (Master), mysql-1 (Slave), mysql-2 (Slave) 
vs 
${deployment name}-${random hash} e.g. myapp-c9e93d4e783165d</code></pre>
- Important, a StatefulSet (SS) Pod will only be created if the previous one is "up-and-running" as SS Pods are created in sequential order (from 0 to N). The same occurs with deletion but in reverse order (from N to 0), i.e. delete order starts from last created SS Pod and a Pod can't be deleted if the previous one couldn't be removed (in reverse order). All this mechanisms are in place in oder to protect/preserve data
- 2 Endpoints/Services for SS=each StatefulSet (SS) Pod gets its "own DNS endpoint" from Service, i.e. 1) there is a Service LoadBalancer (same as Deployment) that will address any replica Pod "plus" 2) each Pod will receive its own Service name or own DNS name (this 2nd characteristic differs from Deployment), e.g.  
<img src="https://github.com/paguerre3/kubeops/blob/main/support/49-statefulset-2-endpoints.PNG" width="73%" height="70%">

- Individual service names for SS Pods are made of<pre><code>${pod name}.${governing service domain} 
e.g. mysql-0.svc2, mysql-1.svc2, mysql-2.svc2</code></pre>
- In summary, when Pod restarts, IP address changes but its predictible name (e.g. myslq-1) and its fixed individual DNS name (e.g. mysql-1.svc2) stay! so these 2 characeristics build the complete "sticky" Identity concept that allows PersistentVolume (PV) re-attachment after any re-schedule, i.e. Sticky Identity ensures that each Pod can retain state and its role after re-creation

**NOTE**
> Administrator still needs to do a lot besides k8s help, e.g. configuring the clonning and data synchronization, make remote storage available, managing and back-up. StateFUL applications are not a perfect candidate for containerized environments while stateLESS applications are, therefore scaling stateLESS applications using docker/k8s is simple because they are meant for it


---
# k8s services
- Each pod gets its own ip but they are ephemeral, i.e. after re-creation a new ip is assigned
- Service provides the solution of an "stable" ip that stays after pod crashes
- Service allows "LoadBalancing" in case of Pod replicas, i.e. it receives a request from a client and then it forwards it to one of the replicated Pods instead of calling them individually
- Service is a good abstraction for loose coupling communication within the cluster and ouside of it (e.g. external browser interfaces with an endpoint Service)
- 1=ClusterIP is the "default" Service type (no need to be explicitly defined) that acts as InternalService abstraction with its own internal IP. ClusterIP is considered an InternalService because it can only be accessed "within" the cluster, e.g. it can receive a request from an Ingress/IngressConroller via its InternalService IP:PORT and then it forwards it to the IP:PORT of one of the Pods replicated among worker Nodes (each worker Node receives its own ip-range for assigning ips to Pods)  

**NOTE**
> as mentioned in previous sections, Service communication with the Pod is done through the use of selectors and target ports. Service is usually defined inside Deployment, e.g. [mongo Deployment + InternalService](https://github.com/paguerre3/kubeops/blob/main/mongo.yml). Service <code>port</code> is arbitrary while <code>targetPort</code> must match the port the Container is listening at. Also, notice that when Services are created k8s generates an Endpoint object with the same name as the Service that is responsible for keeping track of which Pods are the member/endpoints of the Service

<img src="https://github.com/paguerre3/kubeops/blob/main/support/50-endpoints.PNG" width="48%" height="30%">

- Multiple-Port Service: It is possible to support the use case of having a Pod holding multiple Containers and under that scenario the ClusterIP/InternalService "must name" the multiple Ports, e.g. mongodb-application (db access) and exporter (used by Prometheus as scraping port for metrics) 
<img src="https://github.com/paguerre3/kubeops/blob/main/support/51-clusterip-multiple-ports.PNG" width="73%" height="70%">

<img src="https://github.com/paguerre3/kubeops/blob/main/support/52-clusterip-multiple-ports.PNG" width="73%" height="70%">

- 2=Headless Service is used when a client wants to communicate to a "single" Pod directly or when Pods want to communicate directly with an specific Pod, i.e. not randomly selected like when ClusterIP forwards its request. Headless Service is used for stateful applications like data bases, e.g. mongodb or mysql, where Pods have its own Identity (fixed predictible name + individual DNS name) used for re-attachment of Persistent Volumes (PV) so data storage is consistent after any re-schedule, i.e. StatefulSet (SS) Pods have their own state and characteristics/role (e.g. master vs slaves and install order) therefore they can't be considered Identical like Deployment Pods that can be named randomly
- In case of Headless Service, a Client needs to figure out the IP address of each Pod and for doing that there are 2 alternatives: Option 1) Api call to k8s Api Server which is inefficient and it makes the Client App "too tied" to k8s Api. Option 2) "DNS Lookup" to discover IP addresses of Pods, i.e. a DNS Lookup for Service normally returns single IP address (ClusterIP address of InternalService) however if during Headless Service creation the ClusterIP is set to None (<code>clusterIP: none</code>) then DNS Lookup returns Pod IP address instead
<img src="https://github.com/paguerre3/kubeops/blob/main/support/53-headless-svc-dns-lookup.PNG" width="73%" height="70%">

- No ClusterIP is assigned in case of Headless Service, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/54-headless-svc-no-clusterip.PNG" width="73%" height="70%">

- Commonly use cases for communicating with a data base offer Headless and ClusterIP services in combination, i.e. usually Client App "readers" talk with ClusterIP Service which balances the load and then only the "writter" Client speaks to the Headless Service for accessing the Master StatefulSet (SS) Pod that allows updates  
<img src="https://github.com/paguerre3/kubeops/blob/main/support/55-headless-svc-and-clusterip.PNG" width="48%" height="30%">

- there are 3 service type attributes, i.e. Service=<code>spec</code>:<code>type: ClusterIP, NodePort or LoadBalancer</code>
<img src="https://github.com/paguerre3/kubeops/blob/main/support/56-service-types.PNG" width="48%" height="30%">

- 3=NodePort creates a Service that is accessible in a "static" Port on each worker Node of a cluster. NodePort is considered an ExternalService because it allows "external" traffic to have access to a "fixed" Port on each worker Node. <code>nodePort</code> value has a pre-defined range (30000-32767). e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/57-node-port.PNG" width="48%" height="30%">

- Worker Node IP:PORT (<code>nodePort</code>) is expossed so an external client like a browser can access to it. When a NodPort is created also a ClusterIP Service is being generated by default therefore the "external" traffic that comes from a NodePort is forwarded to the clusterIP that will then do the last forward to the Pods of the Node, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/58-node-port-ip.PNG" width="73%" height="70%">

<img src="https://github.com/paguerre3/kubeops/blob/main/support/59-node-port-ip-clusterip.PNG" width="73%" height="70%">

- NodePort Service expands to all its worker Nodes in case of forwarding to replica Pods inside the Nodes, i.e. NodePort receives the request and it forwards to any of the worker Nodes of the cluster, e.g.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/60-node-port-across-replica-pods.PNG" width="48%" height="30%">

**NOTE**
> NodePort is insecure as it opens an stable port to be accessed externally
<img src="https://github.com/paguerre3/kubeops/blob/main/support/61-node-port-insecure.PNG" width="48%" height="30%">

- 4=LoadBalancer is an ExternalService that becomes accessible "externally" through a Cloud Providers LoadBalancer, i.e. each Cloud Provider has its own LoadBalancer implementation that will be used when creating a LoadBalancer type, e.g. aws-elb. When a LoadBalancer Service is created also a NodePort and a ClusterIP get generated, i.e. LoadBalancer receives the request and then it forwards to NodePort that then it forwards to ClusterIP that does the last forward to the Pod  
<img src="https://github.com/paguerre3/kubeops/blob/main/support/62-load-balancer.PNG" width="48%" height="30%">

- LoadBalancer definition requires <code>spec</code>:<code>type: LoadBalancer</code> and <code>nodePort: (30000-32767)</code>, i.e. "entry point" is LoadBalancer, then it goes to NodePort, then to ClusterIP and finally to the Pods
<img src="https://github.com/paguerre3/kubeops/blob/main/support/63-load-balancer-spec.png" width="73%" height="70%">

**NOTE**
> LoadBalancer Service is considered an extension of NodePort Service and NodePort Service is considered an extension of ClusterIP Service. Service type details, e.g.

<img src="https://github.com/paguerre3/kubeops/blob/main/support/64-svc-types-details.PNG" width="73%" height="70%">

**NOTE**
> NodePort is intended for testing purposes so "real" use case scenarios implement Ingress or "native" Cloud Providers LoadBalancer for expossing the cluster, i.e.
<img src="https://github.com/paguerre3/kubeops/blob/main/support/65-ingress-or-load-balancer.PNG" width="48%" height="30%">

