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
* in kubernetes from docker if we issue `kubectl get nodes` in host we get docker-for-desktop 
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
* kops only works on mac or linux

### Lecture 12 - Demo: Preparing Kops Install

* with virtualbox installed on our machine we will start a VM on our host using vagrant
* we setup a folder `mkdir ~/workspace/vms/ubuntu` and move in it
* we install vagrant `sudo apt get install vagrant`
* we run `vagrant init ubuntu/xenial64`
* a Vagrantfile gets added in the folder. we run `vagrant up` to launch the vm
* vagrant uses the already installed virtualbox to install a ubuntu/xenial64 box
* the vm has a folder /vagrant that points to ubuntu folder on our host (~/workspaces/vms/ubuntu)
* to see the ssh configuration and private key location we run `vagrant ssh-config` in the same folder
* we run `vagrant ssh` in same folder to connect to the vm

### Lecture 13 - Preparing AWS for kps Install

* we go to [kops github repo](https://github.com/kubernetes/kops) and get the latest release (the kops-linux-amd64 file) with `wget https://github.com/kubernetes/kops/releases/download/1.10.0/kops-linux-amd64` in the ubuntu virtualbox
* we `chmod +x kops-linux-amd64` to make it executable
* `sudo mv kops-linux-amd64 /usr/local/bin`
* we install python pip (we have it) with `sudo apt-get install python-pip`. we need it for (aws cli)
* we install it with `sudo pip install awscli` . we need to update our toolset first `sudo pip install --upgrade setuptools`
* we can now use 'aws' tools. we need an aws account se tin AWS management console at IAM service
* we will use IAM to creatre a user that will be used by kops to deploy kubernetes
* IAM => Users => create USer => give name => programmatic access =>  use existing policy (full admin rights)
* actually kops needs only the following policies as specified in the [aws and kops user guide](https://github.com/kubernetes/kops/blob/master/docs/aws.md)
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```
* we create the user and see the generated key pair
* in our host we run `aws configure` and enter access and secret key
* we dont set region and output format
* the credentials and config file arew stored in '~/.aws/'
* we make sure to give the user full admin permission (not necesary)
* we go to S3 service in AWS to create a bucker and give it a name (all rest default)
* next we need to set DNS. we select Route 53 service from AWS. if we havea domain name reister we can use it. we have one so we choose Domain management => create hosted zone
* we createa domain name using our domain and add in prefix subdomain. in our case k8s.agileng.io and click create
* to make our domain fw traffic to route 53 we need to use the nameservers assigned at our DNS provider in manage DNS we add 4 new NS records for this host (k8s) adding as target the aws nservers

### Lecture 14 - DNS Troubleshooting

* kops need to have dns working properly as it needs to be able to connect to the cluster and start it
* in our host machine we use host 
* we check that names a re properly set `host -t NS k8s.agileng.io`
* they are set but they point to nowhere. no server running
* if we dont own a domain we can create a hosted zone in AWS
* our root domain is still hosted on namecheap dns servers

### Lecture 15 - Cluster Setup on AWS using Kops

* we login in our vm ubuntu box with (being in the Vagrafile folder)
```
vagrant up
vagrant ssh
```
* we download kubectl in the vm `wget https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubectl`
* we move it in usr/local/bin `sudo mv kubectl /usr/local/bin/` and give it execution permissions `sudo chmod +x /usr/local/bin/kubectl`
* we test the command and it works
* we create a new set of keys to be able to login to the cluster `ssh-keygen -f .ssh/id_rsa`
* we get the public key to upload it to our instances `cat .ssh/id_rsa.pub`
* we rename the kops runtime for ease of use `sudo mv /usr/local/bin/kops-linux-amd64 /usr/local/bin/kops`
* we can use kops to create a cluster on aws with 1 master and 2 nodes `kops create cluster --name=k8s.agileng.io --state=s3://kops-state-4213432 --zones=eu-central-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8s.agileng.io`
* everytime we want to chane the cluster state we need to dd the s3 state bucket
```
kops update cluster k8s.agileng.io --yes --state=s3://kops-state-4213432 
kops edit cluster k8s.agileng.io --yes --state=s3://kops-state-4213432 
kops delete cluster k8s.agileng.io --yes --state=s3://kops-state-4213432 
```
* to set the cluster and config it we need to update `kops update cluster k8s.agileng.io --yes --state=s3://kops-state-4213432`
* our cluster is up and running on aws
* its config file is located on the vm `~/.kube/config`
* there we find the pswrd to login to our cluster
* we use kubectl `kubectl get node` to see the nodes on aws
* we will run the same deployment as we run on minikube or our dev machine to test that the aws k8s cluster is working
* kubectl is conneted to our remote cluster. we use `kubectl run hello-minicube --image=k8s.gcr.io/echoserver:1.4 --port=8080` to create adeployment on cluster
* we expose this deployment in the cluster using the dev service NodePort `kubectl expose deployment hello-minicube --type=NodePort`get nodes
* we can connect in this way to every node on the exposed port except the master
* if we run `kubectl get services` we see the running k8s service and the port we can use to access it
* we go to AWS EC2 to see our 3 running instances in our region (zone of choice)
* from our cluster create command we got 2 autoscaling groups. 1 for master and 1 for nodes
* the nodes have a firewall (in EC2 service settings) in security group nbound rules.
* port 22 is open. not the nodeport. we need to add a security rule for it
* we mod the existing security group of the nodes => inbound => edit => add rule => custom tcp for nodeport (31378) and set the IP of the accessing machine 0.0.0.0/0 is for everyone
* we review the inbound rule at EC2 node description and cp the public ip. we hit `3.121.109.149:31378` in our browser and it works
* our first deployment on AWS k8s cluster using kops works
* we delete the cluster to avoid charging `kops delete cluster k8s.agileng.io --yes --state=s3://kops-state-4213432 `

### Lecture 16 - Building Docker Containers

* this is a docker related lecture
* we use docker engine (docker srv) to build containers
* in linux is straightforward
* tutor offers a precofigured Vagrant box with docker installed called *devops-box*
* we ll use our own. we go on and install docker in our box
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker vagrant

```
* we test installation `sudo docker run hello-world`
* to dockerize an app we need 
	* a Dockerfile
* in vagrant box we add a folder /dockerize and add a Dockerfile
* our image will be a simple node.js app
* the Dockerfile
```
FROM node:4.6
WORKDIR /app
ADD . /app
RUN npm install
EXPOSE 3000
CMD npm start
```
* index.js
```
var express = require('express');
var app = express();
app.get('/', function(req,res){
        res.send('Hello World');
});

var server = app.listen(3000, function() {
        var host = server.address().address;
        var port = server.address().port;

        console.log('Example app listening at http://%s:%d',host,port);
});

```
* package.json
```
{
  "name": "myapp",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node index.js"
  },
  "engines": {
  	"node": "^4.6.1"
  },
  "dependencies": {
  	"express": "^4.14.0"
  }
}
```
* we build using `docker build .` in the directory with the files (we could use CI/CD tool like jenkins)
* we run the container `docker run -p 3000:3000 -t 5b734e078680`
* we visit it with `curl localhost:3000` from inside the box and it works

### Lecture 18 - Docker image Registry

* we can run docker images locally with docker run
* if we want to run them in a remote cluster we need to push them to dockerhub and set the deploy command to get the image from the hub
* we need an account and the docker push command.
* first we login `docker login` tag iut and push it
```
docker login
docker tag <imageid> <dockerid>/<imagename>
docker push <dockerid>/<imagename>
```
* rules of cocker container deployment
	* one process per container
	* data in container are not persistent , use volumes

### Lecture 20 - Running first app on Kubernetes

* we ned to define the pods that will run the containers (it can run 1:n containers)
* pods communicate with each other in cluster using local ports
* the pod definition config file is a yaml file
```
apiVersion: v1
kind: Pod
metadata:
 name: nodehelloworld.example.com
 labels:
  app: helloworld
spec:
 containers:
 - name: k8s-demo
   image: achliopa/docker-demo
   ports:
   - containerPort: 3000

```
* we can use kubectl to create the pod on the cluster `kubectl create -f pod-helloworld.yml` in the folder twher the file resides
* Useful commands
	* kubectl get pod : get information about all running pods
	* kubectl describe pod <podname> : describe a pod
	* kubectl expose pod <podname> --port=<portnumber> --name=<exposedname> : expose the port of a pod (it creates a new service)
	* kubectl port-forward <podname> <portnumber> : Port forward the exposed pod port to our local machine
	* kubectl attach <podname> -i : attach to the pod
	* kubectl exec <podname> --command : execute a command on the pod
	* kubectl label pods <podname> mylabel=<newlabel> : add a new label to a pod
	* kubectl run -i --tty busybox --image=busybox --restart=Never -- sh : run a shell in a pod. useful for debugging

### Lecture 21  - Demo: Running first app on kubernetes

* we have already cloned the course repo in our course dir
* we start minikube `minikube start`
* we test that minikube is up `kubectl get node`
* we open /first-app dir at courseRepo
* we check the helloworld pod yaml file `cat helloworld.yml`
* we create the pod in the minikube cluster `kubectl create -f helloworld.yml`
* we use `kubectl describe pod nodehelloworld.example.com` to see the details
* we use port-forward to forward the pod port to the world `kubectl port-forward nodehelloworld.example.com 8082:3000`
* this does the forward till we ctrl+c the process
* in another terminal we test with `curl localhost:8082`
* the proper way is to use a service or loadbalanbcer that it runs in longterm e.g NodePort `kubectl expose pod nodehellorld.example.com --type=NodePort --name nodehelloworld-service` to see the ip we run `kubectl get service` or `minikube service nodehelloworld-service --url` we hit the url in borwser
* minikube address on host is always 192.168.99.100

### Lecture 22 - Demo: useful Commands

* attaqch to our pod `kubectl attach nodehelloworld.example.com`
* exec a command in pod `kubectl exec nodehelloworld.example.com -- ls /app` executes a command in the pod
* we can describe a service as well `kubectl describe service nodehelloworld-service` we see the internal in-cluster url of the service
* we will launch a busybox pod to listen to the service `kubectl run -i --tty busybox --image=busybox --restart=Never -- sh` we get a shell. we dont have curl so we 
`telnet  172.17.0.16 3000
GET /
` and see the html reply

### Lecture 23 - Service with LoadBalancer

* we will set a load-balancer for the first-app
* in real world we need to safely access the app from outside the cluster
* on AWS we can easily add an external Load Balancer
* it will direct traffic to the correct pod in K8s 
* we can run our own haproxy / nginx LB in front of our cluster
* we use a second yaml config file for the load balancer service 'helloworld-service.yml' it of type LoadBalancer which inb AWS is implemented as ELB service (Elastic Load Balancer)

### Lecture 24 - Demo: Service with AWS ELB LoadBalancer

* we log in to the vagrant box
* we clone the course repo in the box
* in first-app dir we see the 2 cofig scripts for pod and for service
* the target port of the load balcer sercie is the pod port (ref by name)
* we use `kubectl create -f first-app/helloworld.yml` and `kubectl create -f first-app/helloworld-service.yml` 
* MAKE SURE TO START A CLUSTER WITH KOPS first
```
kops create cluster --name=k8s.agileng.io --state=s3://kops-state-4213432 --zones=eu-central-1 --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8s.agileng.io

kops update cluster k8s.agileng.io --yes --state=s3://kops-state-4213432
```
* wait for cluster to start...
* run kubectl create when nodes are ready
* in AWS console we see an ELB instance is created (in new accounts we need to create and delete an ELB first)
* LB listens to port 80 and routes traffic to our 2 node  instances at port 30020
* ELB has a DNS name so we can add it in Route53 in our hosted zone adding a record set helloworld.k8s.agileng.io to point to the alias dns ab819b031fd8911e89914029cfdc45df-218049082.eu-central-1.elb.amazonaws.com
* we can use it in our browser to route to our app
* LB has its own security group connected with the cluster security group
* we delete the cluster 'kops delete cluster k8s.agileng.io --yes --state=s3://kops-state-4213432 ' to avoid charges

### Lecture 25 - Recap: Introduction to Kubernetes

* to expost a pod:
	* with command port-forword
	* with a service: nodeport or loadbalancer

## Section 2 - Kubernetes basics

### Lecture 26 - Node Architecture

* a pod can have multiple containers. the containers can communicate to each other using localhost and a port number
* pods within a cluster can communicate with each other. this communication goes over the virtual cluster network. they use service discovery
* the containers in the pods run using docker. each node must have docker installed
* in each node the following services run
	* kubelet : launches the pods getting pod related info from the master node
	* kube-proxy : feeds info about running pods in the node to the iptables (obkect in node)
	* iptables: is a linux firewall routing traffic using a set of rules. rules are updated by kube-proxy
* external comm to cluster is possible with an external service like load balancer. load balancer is publicly available and forwards com to the cluster. it has a list of nodes and traffi is routed to each node's iptable. iptables forward traffic to the pods
* a pod yaml config file has the spec of the container it has. if it has multiple contianers the spec also has multiple container definitions

### Lecture 27 - Scaling Pods

* if our app is *stateless* we can scale it horizontally
* Stateless = our app does not have a state. not writing to local files/ keeping sessions
* so if no pod has a state. our app is stateless
* traditional DBs are stateful. the DB files cant be split over multiple instances
* web apps canbe stateless. (session management needs to be done out of the container). do not store user data in the container (use redis or cache)
* any files that need to be saved cant be saved locally on the container (e.g S3)
* Stateless app = if we run it multiple times it does not change state
* To run stateful apps we can use volumes. these apps cannot horizontally scale. we can scale them in a single container. but allocate more resources.
* Scalling in Kubernetes can be done using the Replication Controller
* Replication Controiller ensures a specified num of pod replicas run at all time
* A pod created with the replica controller will automatically be replaced if they fail, get deleted or is terminated. is good if we want to make sure uptime
* to replicate an app 2 times istead of pod config we write a repcontroller config. template has the pod definition
```
apiVersion: v1
kind: ReplicationController
metadata:
	name: helloworld-controller
spec:
	replicas; 2
	selector:
		app: helloworld
	template:
		metadata:
			labels:
				app: helloworld
		spec:
			containers:
			- name: k8s-demo
			  image: wardviaene/k8s-demo
			  ports:
			  -	contianerPort: 3000
```

### Lecture 28 - Demo: Replication Controller

* we start our minikube cluster
* our config is in ./CourseRepo/replication-controller/helloworld-repl-controller.yml
* we use the config to create a rep controller in the cluster `kubectl create -f ./CourseRepo/replication-controller/helloworld-repl-controller.yml
`
* we check `kubectl get pods` and see the 2 pods. we see the controller with `kubectl get rc`
* if we describe one of them `kubectl describe pod <pod name>` we see it has a controller attached
* we delete a pod and it is restarted
* we can overwrite the replicas number with the command `kubectl scale --replicas=4 -f ./CourseRepo/replication-controller/helloworld-repl-controller.yml`
* if we get the controller name with `kubectl get rc` we get `kubectl scale --replicas=1 rc/helloworld-controller`
* Replication controller is useful for Horizontal Scalling. in Stateless Apps
* we can delete the rc by name `kubectl delete rc/helloworld-controller`

### Lecture 28 - Deployments

* *Replication Set* is the next-gen Replication Controller
* it supports a new selector that can select based on filtering of a set of values e.g "environment" to be either "qa" or "prod"
* Replication Controller selects only based on equality ==
* The Deployment kubernetes object uses Replication Set
* A *Deployment* declaration in Kubernetes allows us to do app deployments and updates
* The Deployment object defines app state
* Kubernetes then makes sure our cluster matches with the desired state
* Deployment object gives many possiblities over Replication Set or Controller. We can:
	* Create a dployment
	* Update a Deployment
	* Do rolling updates (zero downtime deployment in steps)
	* Roll back to prev deployment
	* Pause/Resume a deployment (to roll-out only a certain %)
* example deployment config yaml file
```
appVersion: extensions/v1beta1
kind: Deployment
metadata:
	name: helloworld-deployment
spec:
	replicas: 3
	template:
		metadata:
			labels:
				app: helloworld
		spec:
			containers:
			- name: k8s-demo
			 image: wardviaene/k8s-demo
			 ports:
			 - containerPort: 3000
```
* spec says 3 replicas of the speced pod
* Useful deployment commands
	* kubectl get deployments : get info on current deployments
	* kubectl get rs : get info on replica sets used by deployments
	* kubectl get pods --show-labels : get pods and also labels attached to the pods
	* kubectl rollout status deployment/<deploymentname> : get deployment status
	* kubectl set image deployment/<deploymentname> <containername>=<imagename>:<versionnum> : run <containername> with the image <imagename> version <versionnum>
	* kubectl edit deployment/<deploymentname> : edit the deployment object
	* kubectl rollout history deployment/<deploymentname> : get the rollout history
	* kubectl rollout undo deployment/<deploymentname> : rollback to previous version
	* kubectl rollout undo deployment/<deploymentname> --to-revision=n: rollback to revision n