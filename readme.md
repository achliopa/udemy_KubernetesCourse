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
* external comm to cluster is possible with an external service like load balancer. load balancer is publicly available and forwards com to the cluster. It has a list of nodes and traffic is routed to each node's iptable. iptables forward traffic to the pods
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

### Lecture 29 - Deployments

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

### Lecture 30 - Demo: Deployment

* in CourseRepo there is a deployment/helloworld.yml yaml file with a deployment config
* we start minikube and deploy it with `kubectl create -f deployment/helloworld.yml`
* we see the deployment with `kubectl get deployments`
* we can see the replica sets inthe deployment with `kubectl get rs`
* we see pods w/ labels with `kubectl get pods --show-labels`
* we see rollout status with `kubectl rollout status deployment/helloworld-deployment`
* we expose deployed pod to outside world `kubectl rollout status deployment/helloworld-deployment`
* we get service url with `kubectl describe service helloworld-deployment`
* to get external url `minikube service helloworld-deployment --url` we visit it and see the message in browser
* we set a new image to deployment with a command `kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2`
* we see the rollowut status and see the deployment history `kubectl rollout history deployment/helloworld-deployment` and see the hoistory. 
* in order to see the changes in history when we deploy with kubectl create -f we should use the flag --record
* to undo last deployment `kubectl rollout undo deployment/helloworld-deployment`
* deployment history keepsonly last 2 revisions. to change that we need to mod the deployment config file. in spec: we need to add revisionHistoryLimit: 100
* to edit changesusing vim we `kubectl edit deployment/helloworld-deployment`
* we set again teh image to add more revisions

### Lecture 31 - Services

* pods are dynamic, they can come and go in the cluster
* if we use a Replication Controller, pods are terminated and created during scaling operations
* in Deployments, when updating the image version, pods are terminated and replaced
* Pods should never be accessed directly but only through Services
* Service is the logical bridge between pods and users or other serivces
* kubectl expose creates a service
* A service creates an endpoint for a pod. Service types that create Virtual Ip for pods are:
	* ClusterIP: a virtual IP address only reachable within the cluster
	* NodePort: a port same on each node and reachable externally (not for production)
	* LoadBalancer: it is created by the cloud provider that will route external traffiv to every node on the NodePort
* There is possibility to use DNS names:
	* ExternalName: if added to service definition provides a DNS name for the service (e.g for service discovery with DNS)
	* We must enable DNS add-on to use it
* Service yaml file definition sample
```
apiVersion: v1
kind: Service
metadate:
	name: helloworld-service
spec:
	ports:
	- port: 31001
	  nodePort: 31001
	  targetPort: nodejs-port
	  protocol: TCP
	selector:
		app: helloworld
	type: NodePort
```
* in NodePort nodePort usually is autoassigned
* by default service runs between ports 30000-32767. this can change adding --service-node-port-range= argument in the kube-apiserver (init scripts)

### Lecture 32 - Demo:Services

* with minikube running
* from first-app in CourseRepo we create a pod with helloworld.yml file `kubectl create -f first-app/helloworld.yml `
* we describe the pod `kubectl describe pod nodehelloworld.example.com` its port is 3000 which is named nodejs-port in the defin yaml file. we export it with the service in helloword-nodeport-service.yml `kubectl create -f first-app/helloworld-nodeport-service.yml`
* we find teh url of the service `minikube service helloworld-service --url`
* CLUSTER-IP is a virtual ip available only in the cluster. if we dont define it inthe YAML file the ip is not static

### Lecture 33 - Labels

* Labels: key/value pairs that can be attached to objects
* used to tag resources. like tags in cloud providers
* we can label our objects. e.g a pod, follwoing an organizational struct e.g key: environment - value: dev / staging/ qa/ prod key: department - value: eng/fin/mark
* labels are not unique. multiple labels can be  added to one object
* once attached to the object. we can use them in filters (Label Selectors)
* in Label Selectors we can use matching expresions to match labels. e.g a pod can only run in a node labeled environment == development. (tag the node, use nodeSelector in pod config)
* we can tag nodes or services.
* to tag nodes we can use kubectl `kubectl label nodes node1 hardware=high-spec`
* use labels  in pod definition
```
nodeSelector:
	hardware: high-spec
```

### Lecture 34 - Demo: NodeSelector using Labels

* we use minikube. nodeselector makes sense in m,ulti node clusters
* in COurseRepo in deployment/ he have a pod definition 'helloworld-nodeselector.yml' that has a nodeselector section added to the deployment config. the containers can run oinly in nodes labeled as high-spec
* we create a deployment with it `kubectl create -f deployment/helloworld-nodeselector.yml` our pods are pending as our node (minikube) is not taggged as high-spec
* we add a label to node programmatically `kubectl label nodes minikube hardware=high-spec` our pods are running now

### Lecture 35 - Healthchecks

* helathchecks are need for production apps
* if our app malfunctions, the pod and container can still be running but the app might not work anymore.
* to detect and resolve problems with the app we can run healthchecks
* there are 2 types of healthchecks
	* run a command in the container periodically
	* periodic checks on a URL (HTTP)
* a production app with a loadbalancer shold always have healthchecks implemented in a way to ensure availability and resiliency
* to add a healthcheck in a pod config
```
livenessProbe:
	httpGet:
		path: /
		port: 3000
	initialDelaySeconds: 15
	timeoutSeconds: 30
```
* usually we use a special url path for healthchecks

### Lecture 36 - Demo: Healthchecks

* in CourseRepo/deployment there is a helloworld-healthcheck.yml deployment config file w/ health checks added
* we run it in minikube with `kubectl create -f deployment/helloworld-healthcheck.yml`
* to see the pods `kubectl get pods`
* if we describe a pod `kubectl describe pod helloworld-deployment-77664648b9-9jgd2` we see the liveness http link
* if the liveness check fails (k8s runs it periodically every specifdied seconds) the pod restarts
* to change the liveness settings we run `kubectl edit <deployment config yaml file>`

### Lecture 37 - Readiness Probe

* apart from livenessProbes we can use readinessProbes on a container within a Pod
* livenessProbes: indicates if a container is running, if check fails the container is restarted
* readinessProbes: indicates if the container is ready to serve requests. if check fails container is not restarted but the POD Ip address is removed from Service, so it does not serve  requests anymore
* readiness tests makes sure that at startup the pod only gets traffic if the test succeeds
* both probes can be used togeether. custom tests can be configured. 
* if container exits when sthing goes wrong we dont need liveness probe.

### Lecture 38 - Demo: Liveness and Readiness Probe

* we can chain commands with && `kubectl create -f helloworld-healthcheck.yml && watch -n1 kubectl get pods` runs deployment AND monitors the creation by attaching to the get pods output
* pods are stated immediantely (no check at startuP)
* we run both probes with a deployment config file `kubectl create -f helloworld-liveness-readiness.yml && watch -n1 kubectl get pods` where the pod startup goes at running state but is not ready at once as checks run
* the config file is
```
          containerPort: 3000
        livenessProbe:
          httpGet:
            path: /
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30
```
* using same url for liveness and readiness will cause a restart even at startup

### Lecture 39 - Pod State

* we will see the different pod and container states
	* Pod Status: high level status (STATUS in kubectl get pods list)
	* Pod Condition: condition of the pod
	* Container State: state of container
* Pod Status: STATUS
	* Running: pod is bound to a node, all containers have been created, at least one container is still running or starting/restarting
	* Pending: Pod is accepted but not running (e.g container image downloading, pod cannot be scheduled because of resource constrains)
	* Succeeded: all containers in the pod have terminated successfully and wont be restarted
	* Failed: all containers in pod terminated and >= container returned failure code
	* Unknown: pod state cant be determined (eg network error)
* Pod Conditions: we can see them with kubectl describ epod. they are conditions the pod has passed
	* Initialized: initialization containers have started successfully
	* Ready: pod can serve requests and will be added to matching services
	* PodScheduled: pod has been scheduled to a node
	* Unschedulable: pod cannod be scheduled *e.g resource constrains)
	* ContainersReady: all containers in pod  ready
* Container State: we see it with `kubectl get pod <PODNAME> -o yaml` as containerStatuses
	* this is docker report
	* Possible Container states: Running, Terminated, Waiting

### Lecture 40 - Pod Lifecycle

* when we launch a Pod:
	* init container runs (if it is secified in pod config). it does some setup (e.g in volumes) PodScheduled=True
	* main container runs (after init container if any  termiantes) Initialized=True
	* main contaner has 2 hookd for scripts: ost start hook and prestop hook
	* in main container run after an initialDelaySeconds probes run Ready=True

### LEcture 41 - Demo: Pod Lifecycle

* in CourseRepo we cd in pod-lifecycle
* in the deployment config file we set the init container pobes and hooks with custom commands
```
kind: Deployment
apiVersion: apps/v1beta1
metadata:
  name: lifecycle
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: lifecycle
    spec:
      initContainers:
      - name:           init
        image:          busybox
        command:       ['sh', '-c', 'sleep 10']
      containers:
      - name: lifecycle-container
        image: busybox
        command: ['sh', '-c', 'echo $(date +%s): Running >> /timing && echo "The app is running!" && /bin/sleep 120']
        readinessProbe:
          exec:
            command: ['sh', '-c', 'echo $(date +%s): readinessProbe >> /timing']
          initialDelaySeconds: 35
        livenessProbe:
          exec:
            command: ['sh', '-c', 'echo $(date +%s): livenessProbe >> /timing']
          initialDelaySeconds: 35
          timeoutSeconds: 30
        lifecycle:
          postStart:
            exec:
              command: ['sh', '-c', 'echo $(date +%s): postStart >> /timing && sleep 10 && echo $(date +%s): end postStart >> /timing']
          preStop:
            exec:
              command: ['sh', '-c', 'echo $(date +%s): preStop >> /timing && sleep 10']

```

* we deply the file `kubectl create -f lifecycle.yaml` while watching for pod status `watch -n1 kubectl get pods`
* we see the logs with `kubectl  exec -it lifecycle-7d55f6f67-zt2vt  -- cat timing
` timining is the file where custom commnads in the yaml config output their logs
```
1544980284: Running
1544980284: postStart
1544980294: end postStart
1544980325: readinessProbe
1544980327: livenessProbe
1544980335: readinessProbe
1544980337: livenessProbe
1544980345: readinessProbe
1544980347: livenessProbe
```

### Lecture 42 - Secrets

* Secrets provide a way in k8s to deistribute credentials, keys, paswords or 'secret' data to pods
* kubernetes itself uses Secrets to provide credentials to access the internal API
* we can use Secrets to provide secret data to our app
* Secrets are Kubernetes native. there are other ways the contianer can get its secrets eg. external vault services
* Secrets can be used for:
	* As environment vars
	* As a file in a pod (this setup uses volumes with files mounted to the container). it can be used for dotenv files or standard files the app reads
	* Use an external image to pull secrets (from aprivate image registry not dockerhub)
* First we must generate the secrets
	* to generate secrets using files
	```
	echo -n 'root' > ./username.txt
	echo -n 'password' > ./password.txt
	kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
	>> secret 'db-user-pass' created
	```
	* a secret can be an SSH key or SSL certificate
	```
	kubectl create secret generic ssl-certificate --from-file=ssh-privatekey=~/.ssh/id_rsa  --sl-cert-=mysslcert.crt
	```
	* we can generate secrets using yaml definitions e.g 'secrets-db-secret.yaml'
* typically sensitive data are used in yaml config files as base64 strings
```
apiversion: v1
kind: Secret
metadata:
	name: db-secret
type: Opaque
data:
	password: cm9vdA==
	username: cGFzc3dvcmQ=
```
* to make base64 strings
```
echo -n "root" | base64
>> cm9vdA==
echo =n "password" | base64
>> cGFzc3dvcmQ=
```
* after creating the yml file we can use kubectl create
```
kubectl create -f secrets-db-secret.yml
```
* after secrets are created we can use them
* if we want to use secrets as env variables we can use a pod to expose them as env vars
* its config will contains in the container spec
```
env:
	- name: SECRET_USERNAME
	  valueFrom:
	  secretKeyRef:
	  	name: db-secret
	  	key: username
	 - name: SECRET_PASSWORD
	 ...
```
* in Secrets vals are stored a s key value pairs
* we can provide also the secrets in a file
```
volumeMounts:
- name: credvolume
  mountPath: /etc/creds
  readOnly: true
volumes:
- name: credvolume
  secret:
  secretName: db-secrets 
```
* key value pairs will be put in the volume mount path as /etc/creds/db-secrets/username and /etc/creds/db-secrets//password 

### Lecture 43 - Demo: Credentials using Volumes

* we use CourseRepo/deployment/helloworld-secrets.yml in minikube for deployment
```
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
data:
  username: cm9vdA==
  password: cGFzc3dvcmQ=
```
* we also deploy a secrets withg volumes deployment config file 'helloworld-secrets-volumes.yml'
```
apiVersion: extensions/v1beta1
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
        - name: nodejs-port
          containerPort: 3000
        volumeMounts:
        - name: cred-volume
          mountPath: /etc/creds
          readOnly: true
      volumes:
      - name: cred-volume
        secret: 
          secretName: db-secrets
```
* we describe a running pod. it has 2 volumes one for our secrets and one for k8s cluster secrets 
* we launch a shell in one of pods `kubectl exec  helloworld-deployment-6d4c6b79d9-8kl5j -it -- /bin/bash` if we look for username and passowrd in volume we see their values ` cat /etc/creds/username`
* we can look in all mounting poionts with `mount` and ls to see the dat ainside
* we exit shell with `exit`

### Lecture 44 - Demo: Running Wordpress on Kubernetes

* data in this pod wont be persistent
* the first yaml config we apply in cluster is is wordpress/wordpress-secrets.yml that sets the secret (db password)
* then we apply the deployment config wordpress/wordpress-single-deployment-no-volumes.yml
	* it uses a wordpress image
	* we provide needed env variables. db passowrd is get from secrets, the others are set manualy
	* pod contains db and wordpress
* the app cannot scale vertically
* we set a service described in wordpress-service.yml to access the pod (nodeport)
* we get the url ` minikube service wordpress-service --url` and visit it.
* our app is functional but data are non-persistent

### Lecture 45 - WebUI

* Kubernetes comes with a WebUI we can use instead of kubectl commands
* we can use it to:
	* get an overview of running apps in cluster
	* create and modify k8s resources and workloads (like kubectl delete and create)
	* get info on state of resources (like kubectl describe)
* we can access WebUI in https://<k8s master>/ui
* if it is not enabled in our deploy type we can install it manually with `kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml`
* if we use minikube we can access it with `minikube dashboard` to see the urrl `minikube dashboard --url`

### Lecture 46 - Demo: WebUI in Kops

* in CourseRepo/dashboard folder it has a readme and sample-user.yml
* readme has instructions how to start dashboard in kops
* in vagrant vm we ssh
* first we need to create a cluster with kops
* we follow the README instructions
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
* we apply the sample-user.yaml to create a serviceAccount `kubectl create -f sample-user.yaml`
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
```
* we give him cluster admin role
* we get the login token `kubectl -n kube-system get secret | grep admin-user`
* we describe the decret to see the token `kubectl -n kube-system describe secret admin-user-token-4z8hl`
* we use this token to login to the cluster without using the certificates
* cerificates is the standard way of kops to accesst he cluster
* our certifocates are stored in ~/.kube/config
* we also need a password. we see it in `kubectl config view`
* we will use it to llogin to the api server 'https://api.k8s.agileng.io'
* we use 'admin' as user and the password we just saw
* being logged in we see all the available paths
* to access the ui we write 'https://api.k8s.agileng.io/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login'
* we select token and cp the token we saw earlier
* we are in the dashboard at last of our AWS kubernetes cluster


### Lecture 47 - Demo:WebUI

* in WebUI we can see all sorts of info for the cluster
* we can create a deployment using the UI
* we kreate a deployment helloworld.yml and see the pods in UI

## Section 3 - Advanced Topics

### Lecture 48 - Service Discovery

* Since Kubernetes 1.3 DNS is a built-in service launched automatically using the addon manager.
* It can be used for service discovery using DNS
* the addons are in /etc/kubernetes/addons dir on master node
* DNS service can be used within pods to find other services running on the same cluster
* containers in a pod dont need it. the communicate with each other direclty with localhost:port
* to make DNS work a pod will always nedd a Service definition
* each pod gets its own ip in the cluster from the attached service. the service name attached to the pod is used by DNS so that apps in pods can communicate with each other
* `host app2-service` command returns the IP address of app2 serivce
* DNS discovery using serivce name works if both are in the same namespace
* `host app2-serivce.default` looks in default namespace for the service
* the full dns on a cluster node is `host app2-service.default.svc.cluster.local` it resolves in the IP address
* how DNS works in cluster??? to see it we lauch a dummy pod `kubectl run -i -tty busybox --image=busybox --restart=Never -- sh`. 
* in the containers shell we run `cat /etc/resolv.conf`
* what we get are the credentials (IP address) on the DNS nameserver. the kube-dns serivce running (DNS Server) in the kube-system namespace. to see it in get pods we need the flag -n kube-system

### Lecture 49 - Demo: Service Discovery

* in CourseRepo we see the folder service-discovery
* it has a secrets.yml filesetting a num of key value pairs like username password and database using base64 strings
* we apply it in minikube on host `kubectl create -f service-discovery/secrets.yml`
* database.yml creates a database pod (mysql) and database-service.yml a db servoce for the pod. we apply them both on minikube. service exposes a port with nodeport
* we apply a hellowworld-db.yml deployment for a node,js app that connects to db
* we use `command: ["node", "index-db.js"]` to execute an alternate command in standard tutors image
* it also sets a num of env vars based on secrets to connect to db
* MYSQL_HOST env var is of particular insterest as its value is the service-name. this needs DNS service so that node.js will connect to db service by name
* we createa nodeport service for the node.js helloworld pod
* we get the url at `minikube service helloworld-db-service --url` open it with browser and SUCCESS
* we connect to the bg container `kubectl exec database -i -t -- mysql -u root -p` pswd is rootpassowrd
```
mysql> show databases;
mysql> use helloworld;
mysql> show tables;
mysql> select * from visits;
mysql> \q
```
* to test dns we log in to a busybox container `kubectl run -i --tty busybox --image=busybox --restart=Never -- sh`. busybox has nslookup we use it to test dns `nslookup helloworld-db`
* it fails as we need to set 'default.svc.cluster.local ' in /etc/resolf.conf
* we try `nslookup -type=a database-service.default.svc.cluster.local` and it works
* better use host. more stable

### Lecture 50 - ConfigMap

* 