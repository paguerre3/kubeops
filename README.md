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
### kubernetes components
- Node or worker node is a simple server that contains one or more pods in it, e.g. node1=physical or virtal machine. Physical machine layers=hardware+os kernel+application vs virtual machine=os kernel+application vs virtual container=works at application layer as its faster because runs on top of host os kernel and smaller, e.g. being os kernel=linux and application=alpine or ubuntu
- Pod is an abstraction over a container considered as the smallest unit of k8s, e.g. pod1=abstraction-over-docker-container-of-java-app and pod2=abstraction-over-docker-container-of-mongo-db inside node1/server. A pod creates a running environment on top of a container because k8s abstracts the running environment of the container technology so it isn't coupled to an scpecific technology as docker/docker-runtime, i.e. you only interact with kubernetes layer and the underlaying container technology can be replaced if needed

**NOTE**
> Usually 1 containerized application per pod
- k8s offers an out of the box virtual network which means that each pod inside a node has its own IP, i.e. each Pod gets its own IP address (not the container) so each pod communicates with another pod inside the same node using the internal IP address (not external IP address)
![Screenshot|512x397,20%](https://github.com/paguerre3/kubeops/blob/main/support/1-pod-communication.PNG) 