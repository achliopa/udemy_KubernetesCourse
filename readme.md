# Udemy Course - Learn Devops: The complete Kubernetes Course

* [Course](https://www.udemy.com/learn-devops-the-complete-kubernetes-course/)
* [Repository]()

## Installation Procedure

* course github repository scripts are updates to the lates K8s version
* k8s are updated every 3months
* install kubectl (see docker_kubernetes)
* install minikube (see docker_kuberenetes)
* install virtualbox (see docker_kubernetes)
* start minikube cluster `minikube start`
* we do a test deploying a demo app using imperative commands
```
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
minikube service hello-minikube --url
```
* we test in browser

## Section 1 - Introduction to Kubernetes

### Lecture 4 - Kubernetes Introduction

* Kubernetes: Opensource orchestration system for Docker containers
	* allow scheduling of containers on a node cluster
	* allows running multiple containers on a node
	* allows running long running services
	* manages containers state. restarts them. moves them to another node
* A Kubernetes cluster can scale to 1000s of nodes
* Other docker orchestrators:
	* Docker Swarm (not so extensible)
	* Mesos (for other types of containers)
Kubernetes can run:
	* on-premise (private datacenter)
	* on cloud
	* hybrid: private & public

### Lecture 5 - Containers Introduction

* VMs a re heavy and take time to bootup
* Containers are lightweight. only what is necessary for the app. boots up fast
* On cloud providers conteiers run on a guest OS to separate users data
* Docker = Docker Engine (runtime)
* Docker Benefits:
	* Isolation (binary_+ dependencies)
	* Closer relation between dev,test,prod envs
	* Docker enables faster deploy cycles
	* You can run the images anywhere
	* Docker uses linux containers (kernel feat) for os-level isolation

### Lecture 6 - Kubernetes Setup

* kubernetes can run anywhere
* more integrations for Cloud Providers (AWS,GCE)
* Volumes and External Load balancers work only on supported Cloud providers
* minikube is the simplest way to run a single node cluster on a host machine
* we ll spin a cluster on AWS with kops

### Lecture 7 - Local setup with Minikube

* minikube runs a single node k8s cluster on a linux vm (virtualbox)
* no high availability, not production grade 
* start it with `minikube start`

### Lecture 8 - Demo: Minikube

* get latest release from [github](https://github.com/kubernetes/minikube)
* follow installation instructions
* minikube runs from /usr/local/bin
* 'minikube start' to start the cluster
* we can see the cluster config file at '~/.kube/config' where we can see certificate location and ip
* we install kubectl
* kubectl looks by default to the ~/.kube/config file and knows how to connect to minikube cluster
* we issue imperative commands to cluster with kubectl
* `kubectl run` deploys a docker image as a service `kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080` deploys an image in the cluster (pod) and sets a contanier port
* `kubectl expose deployment hello-minikube --type=NodePort` exposes the pod to outside using a nodeport servce (good only for development and local deploys)
* this is the imperative way of issuing commands which is not recommneded. we sould write and apply config yaml files instead
* to find how we will access the service from external machine we issue `minikube service hello-minikube --url` get the url. and visit it with browser to see the infamous hello-world container running
* to stop our cluster `minikube stop`

### Lecture 9 - Installing Kubernetes using the Docker Client

* we ll see how to install kubernetes locally using the docker client
* we use it ONLY IF minkube does not work
* new docker versions allow installing k8s with docker client
* it is enabled from docker CE executable in MAC and Windows. not available in Linux
* it starts a standalone kubernetes cluster
* in kubernetes from docker if we issue `kubctl get nodes` in host we get docker-for-desktop 
* if we run also minikube we ll see 2 nodes and 2 contexes if we issue `kubectl config get-contexts`
* to select between multiple context we use `kubectl config use-context <context name from list>`

### Lecture 10 - Minikube vs Docker Client vs Kops vs Kubeadm

* there are multiple tools to install a k8s cluster
* minikube and docker client are for local installs (single node)
* For production clusters we need other tools like *Kops* and *kubeadm*
* on AWS the best tool is kops. AWS is working on EKS(hosted kubernetes). this will be a better option (no need to maintain masters)
* For other types of installs or if we cannot malke kops work, we can use kubeadm
* kubeadm is not recommneded for AWS. in AWS we have automatic integration with kops

### Lecture 11 - Introduction to kops

* kops = Kubernetes operations
* it is used to setup k8s on AWS
* the tool allows to do production grade k8s installations, upgrades and management