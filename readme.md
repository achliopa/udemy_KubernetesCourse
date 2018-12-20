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

* ConfigMap object is used for config params that are not secret
* the input is like Secrets key-value pairs
* ConfigMap key-value pairs can be read by the ap as
	* Environment variables
	* Container commandline arguments in the Pod config
	* through volumes
* ConfigMap can contain full config files (standard files not k8s coonfig YAML)
	* a webserver config file
	* the file can be mounted with volumes where the application expects the config file
	* in that way we can inject config settings in the contianers without moding the container
* a sample workflow... we put the stdin in app.properties file (till we write EOF) and then add the file to configMap
```
cat <<EOF > app.properties
driver=jdbc
database=postgres
lookandfeel=1
otherparams=xyz
param.with.hierarchy=xyz
EOF
kubectl create configmap app-config --from-file=app.properties
```
* To Use a ConfigMap we can create a pod that exposes the Configmap using a volume
```
...
volumeMounts:
- name: config-volume
  mountPath: /etc/config
volumes:
- name: config-volume
  configMap:
  name: app-config
```
* config values in volume configMap are a ccessible as paths: param.with.hierarchy => /etc/config/param/with/hierarchy
* also we can create a Pod that exposes ConfigMap as environment vars
```
...
env:
 - name: DRIVER
   valueFrom:
   	configMapKeyRef:
   	 name: app-config
   	 key: driver
 - name: DATABASE
 ....
```

### Lecture 51 - Demo: ConfigMap

* in CourseRep we have a folder configmap. in there we have an nginx conf file named reverseproxy.conf
* we make a configmap out of it in minikube with `kubectl create configmap nginx-config --from-file=reverseproxy.conf` we find it as `kubectl get configmap` and we can see its configuration as YAML file (SWEETTT!!!) with `kubectl get configmap <NAME> -o yaml`
* our pod config is in nginx.yml. it uses  the configmap as volume
* the mount path is where nginx expects its configuration (/etc/nginx/conf.d). not a random path
* we apply it and its service as well (nginx-service.yml). grab its url and visit it
* -vvvv in curl adds verbosity `curl http:/100:31007 -vvvv`
* `kubectl exec -i -t helloworld-nginx -c nginx -- bash` to login the container shell. we view the nginx process

### Lecture 52 - Ingress Controller

* Ingress is a solution in Kubernetes >v1.1 that allows inbound connections to the Cluster
* It is an alternative to Loadbalancer and nodePorts (good for cloud providers without load balancer)
* Ingress allows easily expose services that need to be accessed from outside
* With ingress we can run our own ingress controller (a loadbalancer) within the K8s cluster
* There are default ingress controllers but we can write our own
* When we hit the cluster from outside
	* Ingress Controller Serivce listens to port 80 and 443
	* It is attached to a Pod with an ingress controller container (typically nginx ingress controller)
	* It directs traffic to the app containers in their pods (through their service)
	* trffic routing is done based on rules we add to the ingress controller (e.g host1.example.com -> pod1)
* We create the ingress rules using the Ingress object
* for routing we can use path or hostname or both

### Lecture 53 - Demo: Ingress Controller

* in CourseRepo we visit folder ingress/
* we see an example ingress controller config yaml 'nginx-ingress-controller.yml'
* it is a default ingress controller from nginx. it is a replication controller (it restarts). made by google
* it has probes. it works in port 80 and 443
* default backend service is where traffic is routed when there is no match
* rules are in ingress.yml that specs the ingress object
* we apply ingress.yml (Ingress obj), nginx-ingress-controller.yml (ingress controller), echoservice.yml (default service when there is no match), helloworld-v1.yml (pod app 1), helloworld-v2.yml (pod app 2)
* with `minikube ip`we get cluster ip
* we hardcode host in curl `curl 192.168.99.100 -H 'Host: helloworld-v1.example.com'`
* routing works OK

### Lecture 54 - External DNS

* on public cloud providers we can use the ingress controller to reduce the cost of our loadbalancers
* we can use 1 LoadBalancer that captures all external traffic and send it to the ingress controller (vs using many LBs)
* ingress controller can be configured to route the different traffic to all our apps on cluster based on HTTP rules (host and prefix)
* This works only for HTTP(s) based apps. if we use othe rprotocol we have to use more LBs
* one great tool to enable the ingres approach is External DNS
* this tool will automatically create the DNS records in our external  DNS server (like route53)
* for every hostname we use in ingress it will create a new record to send traffic to uour loadbalancer
* Major DNS providers are supported: Google CloudDNS, Route53, AzureDNS, CloudFlare, DigitalOcean etc
* other setups are posible without ingress controllers (e.g directly on hostport-nodeport).still WorkInProgress
* Ingress example with Route53 and AWS LB.
	* ingress rules are added to the Ingress Serivce object
	* rules are red by nginx-ingress-controller (in the attached pod)
	* external dns container in anohter in-cluster pod reads the rules and adds dns records for the hostnames in the external DNS providers records (out of cluster)
	* Internet client goes to Eternal DNS provider record and gets ip
	* client with ip hits the external loadbalancer
	* loadbalancer sents to ingress (and ingress controller)
	* ingress controller routes to pods based on rules

### Lecture 55 - Demo: External DNS

* in CourseRepo in folder external-dns we have a README
* it contains the commands needed in our cluster to run the demo 
* we start vagrant and create a cluster on AWS with kops
* before we use external dns we should add a policy using the ready made script `vim ./put-node-policy.sh`
* this script adds permmissions to the nodes and allows any pod to use these priviledges
* adding priviledeg to every pod and everynode is not recommended especially as AWS IAM user has full access admin role
* we update NODE_ROLE giving our AWS IAM role name and region
* we apply it running the script in master 
* we apply all configs in ingress folder `kubectl apply -f ../ingress/`
* apply allows to do changes to YAML files and reapply
* in 'nginx-ingress-controller.yml' we still have hostport. we remove both as both ports will be exposed using the AWS load balancer. we reapply
* we apply service-l4.yml which creates a loadbalancer with an aws external ip to connect to the cluster
* we apply external dns service and ingress rule
* we mod the rules in ingress.yml to point to our domain hostnames same for external0dns and service-l4
* we see the logs of the external-dns pod with all the record settings
* SUCCESS

### Lecture 56 - Volumes

* needed for stateful apps
* they allow us to store data outside of the container
* stateless apps dont keep local state. they use an external service (e.g Database, Caching service etc)
* we can run these services inside the cluster using persistent Volumes to store their data
* Volumes can be attached using different volume plugins
* a container can have a local volume (in-node storage) that can be used by another pod in the node
* but volumes can exist outside the node
	* AWS Cloud: EBS Storage
	* Google Cloud: Google Disk
	* MS Cloud: Azure Disk
	* NW Storage: NFS, Cephfs
* using volumes we can deploy apps with state on cluster
* these apps RW to files on local filesystem
* volumes out of the nod emake sense for cloud apps. node can stop running and rescheduled on another node. the new node will see the cloud volume if it is spawned in the same zone
* to use a volume we need to create one first (YAML config) or by command `aws ec2 create-volume size 10 --region us-east-1 --availability-zone us-east-1a --volume-type gp2`
* next we need a pod with a volume definition (volumeMounts)

### Lecture 57 - Demo: Volumes

* we work on AWS cluster. delete previous one create new one
* we need to setup a volume in AWS. we use thje AWS CLI `aws ec2 create-volume --size 1 --region eu-central-1 --availability-zone eu-central-1a --volume-type gp2 --tag-specifications 'ResourceType=volume, Tags=[{Key=KubernetesCluster, Value=k8s.agileng.io}]'`
* volume must be in same zone as cluster
* we use a yaml deployment file for a deployment of the app with volumes helloworld-with-volume.yaml
* the volume must have the same domain as value as the cluster
* we need to set the AWS EBS volumeId
```
 volumeMounts:
        - mountPath: /myvol
          name: myvolume
      volumes:
      - name: myvolume
        awsElasticBlockStore:
          volumeID: # insert AWS EBS volumeID here                                                                                                                
```
* to delete a volume from AWS `aws ec2 delete-volume --volume-id=vol-0cf38a917538ce282 --region=eu-central-1
`
* we deploy the file on cluster. volume is attached AWSElasticBlockStore (a Persistent Disk resource in AWS)
* i execute bash in pod `kubectl exec helloworld-deployment-7d668994d7-hnrwr -it -- bash` and visit moutnpoint `ls -ahl /myvol/` i write sthing and  stop the node. when it restarts the file is there
* to stop a node `kubectl drain ip-172-20-54-9.eu-central-1.compute.internal --force`
* we open a cell on new pod and file is persistent

### Lecture 58 - Volumes Autoprovisioning

* kubernetes plugins can provision storage for us
* The AWS plugin can provision storage by creating volumes before attaching them to a node
* this is done with StorageClas object
* the yaml file for the auto provisioned volumes is 
```
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
	name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
	type: gp2
	zone: eu-central-1
```
* this allows to create volumes with aws-ebs-provisioner
* k8s will provision volumes of type gp2 for us
* thenwe can create avolume claim and specify the size
```
kind: PersistentVOlumeClaim
apiVersion: v1
metadata:
	name: myclaim
	annotations:
		volume.beta.kuberentes.io/storage-class: "standard"
spec:
	accessModes:
	- ReadWriteOnce
	resources:
		requests:
			storage: 1Gi
```
* then we can launch apod using the volume claim
```
...
volumes:
	-name: mypd
	 persistentVolumeClaim:
	 	claimName: myclaim
```

### Lecture 59 - Demo: Wordpress with Volumes

* we create a cluster in AWS
* we go to CourseRepo/wordpress-volume
* thereis a stroageclass config yaml storage.yml . we fix the zone to match the cluster
* pv-claim.yml configs the claim we set it to 1G
* wordpress-db.yml sets the deployment as repl controller
* volumes are partitions of disk so they come with a lost+found folder (we ingore it in mount)
* we use a nodeport servioce wordpress-db-service.yml
* we also config a Secrets object with a some keyvalue pairs
* wordpress-web.yml cas the deployment config for wordpress. it uses the secrets as env vars
* in wordpress we can upload pics. we use volumemount on the path that wp uses for uploads
* we use efs for this storage in AWS an nfs file system
* nfs is not good for DB . we use block storage for that
* we need to create a new efs in AWS first
* wordpress-web-service puts an LB infront of the pods
* we use aws to create efs `aws efs create-file-system --creation-token 2 --region=eu-central-1` token must be unique
* then we need to create an efs mount target using the efs fsid. we also need the subnet id of th ecluster . we get it with `aws ecs decribe-instances` as well as security groups
* the command is `aws efs create-mount-target --file-system-id=fs-18912941 --subnet-id=subnet-00edd71d6c0e510fe --security-groups=sg-05592d74d0de1a8a2 --region=eu-central-1`
* we use the filesystem-id in wordpress-web.yml and set correct region
* we create storage => then the pvclaim => then the secrets => thenb the db => then the db-service
* we check pvs status `kubectl get pvc` the volume is bound
* we describe the pod and it ihas volume mounted on EBS 
* we create wordpress web. it uses efs that takes time to create
* we create the wordpress service (LB)
* to use proper DNS we go to route53, create a new record se tit as alias and point it to new ELB
* visit wordpress and do the installation
* we have 2 pods running for wordpress ans one for db. db data are saved in a volume
* we also have efs for media. if i try to save data i see an error. dir is not writeable. the wordpress image was not built to workj with efs volumes. we can make our own custom image or add a cmd in the deployment config
* we use `kubectl edit deploy/wordpress-deployment` and add in spec containers
```
      - command:
        - bash
        - -c
        - chown www-data:www-data /var/www/html/wp-content/uploads && docker-entrypoint.sh apache2-foreground

```
* i remove the pods and data is there

### Lecture 60 - Pod Presets

* Pod Presets can inject info into pods at runtime
* they are used to inject kubernetes resources (e.g Secrets, ConfigMaps, Volumes, Env vars)
* if we have multiple applications to deploy ans all need a specific credential
	* we can edit each deployments condif adding the credential
	* we can use presetes to create 1 Preset object. this object will inject an env variable or config file to all matching pods
when injecting enviroment vars and volumemounts. the pod preset will apply the changes to all containers within the pod
* preset uses a selector to be applied
* we can use >1 pod presets. in case of conflict they wont be applied
* PodPresets can match >0 pods. it is possible that a match will happen in future when a pod is launched

### Lecture 61 - Demo: Presets

* we lauch the aws cluster from vagrant vm
* we go to kubernetes-course/pod-presets
* in README it says that we need to edit the cluster adding a spec so as to enable PodPresets
```
spec:
  kubeAPIServer:
    enableAdmissionPlugins:
    - Initializers
    - NamespaceLifecycle
    - LimitRanger
    - ServiceAccount
    - PersistentVolumeLabel
    - DefaultStorageClass
    - DefaultTolerationSeconds
    - MutatingAdmissionWebhook
    - ValidatingAdmissionWebhook
    - NodeRestriction
    - ResourceQuota
    - PodPreset
    runtimeConfig:
      settings.k8s.io/v1alpha1: "true"

```
* we edit the cluster adding the spec in the bottom `kops edit cluster k8s.agileng.io --state=s3://kops-state-4213432` enabling a set of settings for presets
* we update the cluster
* PodPresets config YAML is in pod-presets.yaml
```
apiVersion: settings.k8s.io/v1alpha1   # you might have to change this after PodPresets become stable 
kind: PodPreset
metadata:
  name: share-credential
spec:
  selector:
    matchLabels:
      app: myapp
  env:
    - name: MY_SECRET
      value: "123456"
  volumeMounts:
    - mountPath: /share
      name: share-volume
  volumes:
    - name: share-volume
      emptyDir: {}
```
* in deployment YAML there is no spec for presets. the prams are passed based on app name (selector)
* we apply presets and see them with `kubectl get podpresets`
* we apply the deployments and see the params of presets in pods

### Lecture 62 - StatefulSets

* stateful distributed apps on a k8s cluster
* it started as PetSets in v1.3 renamed at StatefulSets. stable sinc v1.9
* it is introduced for stateful apps to ensure a stable pod hostname (instead of podname randomizing)
* podname will have a sticky identity kept in reschedule
* StatefulSets allow stateful apps stable storage with volumesbased on the ordinal number (podname-x)
* deleting or scaling dow a StatefulSet wont delete the volumes associated with the set
* StatefulSets allow the app to use DNS to find peers
* ElasticSearch and Cassandra clusters use DNS to find cluster members
* a dynamic name cannot be used in config files
* StatefulSet allows ordering startup and teardown of app
	* SCALING UP 0 -> n scaling down n -> 0 

### Lecture 63 - Demo: StatefulSets

* we use Cassandra (scalable db) for the demo.
* we start our aws cluster from vagrant master
* in kubernetes-course/statefulset thre is a cassandra.yml
* cassandra is a scalalbe db so the nodes need to find each other by hostname
* the YAML file config the StatefulSet is like a Deployment object
* cassandra needs small not micro nodes to run. it has resource requirements * it uses IPC lock and when stopping it drains the node
* seeds are retrieved by pod 0 by hostname
* it defines a StorageClass and a Service
* we deploy and with pods running we execute `kubectl exec -it cassandra-0 -- nodetool status` we see noide status.
* if we login the pods shell we can ping other pods by name `ping cassandra-1.cassandra`

### Lecture 64 - Daemon Sets

* Daemon Sets ensure that every single node in teh Kubernetes cluster runs the same pod resource
* it is useful if we want to ensure that a certain pod is running on every single k8s node
* when we add a node to the cluster a new pod with be started automatically on the node.
* if the node is removed the pod WONT be rescheduled to another node
* use cases:
	* logging aggregators
	* monitoring
	* LBs / reverse proxies, API gateways
* syntax of config YAML is like a RepicationSet or Deployment

### Lecture 65 - Resource Usage Monitoring

* Heapster enables container cluster monitoring and Performance analysis
* it provides a monitoring platform for K8s
* it is a prereq if we want to do pod auto-scaling in k8s
* Heapster exports cluster metrics via REST endpoints
* we can use different backends with heappster (InfluxDB, Google cloud monitoring, Kafka)
* Visualizations can be shown using Grafana
* once monitoring is enabled K8s dashboard will also show graphs
* Heapster, InfluxDB, Grafana can be started as  pods
* their yaml files are in [gihub](https://github.com/kubernetes-retired/heapster/tree/master/deploy/kube-config/influxdb)
* it is DEPRECATED
* the platform gets deployes by applying the config files or using the addon system
* in app nodes kubelet and cadvisor gather pod metrics. in heapster node heapster pod gathers all metrics and sends them to influxdb pod. grafana pod shows them

### Lecture 66 - Demo:Resource Monitoring using Metrics Server

* since v1.8 Metrics Server replaced Heapster. 
* to show the metrics we can use a 3rd party tool (Prometheus)
* we will see now how to install Metrics Server in teh cluster
* metrics will appear in k8s dashboard
* in our vagrant master node (kops cluster is up and running) we go to kubernetes-course/metrics-server/
* there are a number of YAML files. we apply them all `kubectl create -f .`
* with metrics server running we can run `kubectl top` for  node or pod to show metrics 
* we run a simple pod so that we gather metrics `kubectl run hello-kubernetes --image=k8s.gcr.io/echoserver:1.4 --port=8080`
* Metrics server is required for autoscalling

### Lecture 68 - Autoscaling

* kubernetes has the ability to automatically scale pods based on metrics
* k8s can automatically scale a deployment, replication controller or replicaset
* in k8s v1.3 scaling based on CPU usage is possible out-of-the-box
* with alpha support applcation based metrics are also available (queries/sec avg req latency). to enable it the cluster has to be started with the env var ENABLE_CUSTOM_METRICS to true
* autoscaling will periodically query the utilization for the target pods. default is 30sec. we can change that with --horizontal-pod-autoscaler-sync-period= when launching the controller manager
* autoscaling will use metrics server to gather its metrics and decide on sclaing
* Example
	* a deployment with a pod with CPU resource of 200m (200millicores = 20% of CPU core on teh node)
	* we add autoscalling at 50% if CPU usage (100m)
	* Horizontal pod autoscaling will increase/decrease pods to maintain target CPU utilization of 50%. 
* a pod to use for testing autoscaling
```
...
resources:
	requests:
		cpu: 200m
...
```
* autoscaling YAML config. its is applied on a deployment
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example-autoscaler
spec:
 scaleTargetRef:
   apiVersion: extensions/v1beta1
   kind: Deployment
   name: hpa-example
 minReplicas: 1
 maxReplicas: 10
 targetCPUUtilizationPercentage: 50
```

### Lecture 69 - Demo: Autoscaling

* on our minikube cluster oh host
* we go to COurseRepo/autoscaling/ in hpa-example.yaml a Deployment a Nodeport service and a HPA are configured
* we apply it and see the hpa with `kubectl get hpa`
* to test we run a load-generator busybox containet in a pod and log in to tits shell `kubectl run -i --tty load-generator --image=busybox /bin/sh`
* we hit the hpa-example pod to generate some load `wget http://hpa-example.default.svc.cluster.local:31001
`
* we run it in aloop to create load `while true; do wget -q -O- http://hpa-example.default.svc.cluster.local:31001; done`
* autoscalle sees the load and scales

### Lecture 70 - Affinity/Anti-affinity

* ona previous demo we saw how to use nodeSelector to make sure pods get scheduled on specific nodes
* affinity/anti-affinity feature allows us to do more complex scheduling than the nodeSelector and also work on Pods
	* language is more expressive
	* we can create rules that are not hard requirements, but rather a preffered rule, meaning that the scheduler will still be able to schedule our pod, even if the rules cannot be met
	* we can create rules that take other pod labels into account
* k8s can do node affinity and pod affinity/antiaffinity
* node affinity is like nodeSelector
* pod affinity/antiaffinity allows us to create rules on how pods should be scheduled based on other running pods
* aff/antiaff mech is relevant only in scheduling
* Node affinity types:
	* requiredDuringSchedulingIgnoredDuringExecution (hard req)
	* preferredDuringSchedulingIgnoredDuringExecution (soft req)
* it uses matchExpressions
* preferredDuringSchedulingIgnoredDuringExecution uses weighting. higher weight , more weight given to the rule. the node with higher score gets the pod scheduled
* in addition to custom labels we add to nodes, pre-populated labels also exist.
	* kubernetes.io/hostname
	* failure-domain.beta.kubernetes.io/zone
	* beta.kubernetes.io/instance-type
	* beta.kubernetes.io/os

### Lecture 71 - Demo: Affinity / Anti-Affinity

* we start our cluster on AWS from vagrant vm
* we describe a node and see the pre-populated available labels
* we go to kubernetes-course/affinity/ where he have a node-affinity.yaml file that adds nodeAffinity to Deployment pods
* we create it and then describe pod. they are pending as no node has the required labels
* we add labels `kubectl label node ip-172-20-48-241.eu-central-1.compute.internal env=dev`
* our pods are assigned 
( i create a better match adding a team  label. i stop a pod and its rescehduled in the preferred node of better match

### Lecture 72 - Interpod Affinity and ANti-Affinity

* this mechanism allows us to infuence pod scheduling based on labels of other pods running in the cluster
* pods belong to a namespace. our affinity rules will apply to a namespace. if no namespace is speced it defaults to the pod namespace
* the same 2 types like node affinity arew available: requiredDuringSchedulingIgnoredDuringExecution, preferedDuringSchedulingIgnoredDuringExecution
* a good use case for pod affinity is co-located pods in same node (e.g redis and app pod)
* another usecase is pod co-location in same availability zone
* in pod affinity/anti-affinity rules, we need to spec a toplogy domain (topologyKey). it refers to a node label. if rule matches the pod will only be scheduled on nodes with the same  topologyKey s the current running pod (matching pod). this may lead ohn the pod being scheduled on a different node than the matched pod if ithas the same ropologyKey
* anti-affinity is the opposite. it canbe used e.g to make sure a pod is scheduled once on a node
* based on the matching rule. it will not scheule the pod on a node with the same topologyKey value as the matching pod node
* rules operators:
	* In,NotIn (does a label has one of the values)
	* Exists, DoesNotExist (does a label exist or not)
* affinity requires a lot of processing power

### Lecture 73 - Demo: Interpod Affinity

* wee are in vagrant vm with our AWS cluster running
* in kubernetes-course/affinity we see pod-affinity.yaml. it has 2 deployment one with no rules and one with affinity rules based on the other deployment pods
* we apply it. both go to same node `kubectl get pod -o wide`
* we scale the replicas `kubectl scale --replicas=4 deployment/pod-affinity-2`. still all go to same node
* we delete deployments `kubectl delete 0f pod-affinity.yaml` we see the node labels with `kubectl describe node |less`
* we change the topologyKey to a more geenric label like zone. no pods have more scheduling flex. all nodes are in same zone

### Lecture 74 - Demo: Interpod Anti-Affinity

* same as before opposite logic

### Lecture 75 - Taints and Tolerations

* tolerations is the opposite of node affinity. they allow a node to repel a set of nodes
* taints mark a node. tolerations are applied to pods to influence the scheduling of pods
* one usecase for taints is to make sure that when we create pods they are not scheduled on the master node
* in clusters of >1 nodes master gets a taint from k8s. 'node-role.kubernetes.io/master:NoSchedule'. we can only schedule on master when we have a toleration applied to a pod. by default no tolerations are applied
* to add a taint to a node `kubectl taint nodes <nodename> key=value:NoSchedule` this makes sure that no pods will be scheduled on <nodename> node as long as they dont have a matching toleration like the following
```
...
tolerations:
- key: "key"
  operatior: "Equal"
  value: "value"
  effect: "NoSchedule"
```
* we can use as operatiors: Equal(provide key-value) Exists(provide key only)
* taints can also be a preference (soft req)
	* NoSchedule: hard req
	* PreferNoSchedule: soft req
* taint takes effect only at scheduling unless we use: NoExecute (evict non matching pods). with NoExecute we can spec also the time till eviction (tolerationSeconds) which we put in toleration. if we dont put time toleration will match and there will be no eviction after time (tolerated)
* use cases:
	* taints for master nodes
	* taint nodes dedicated to teams or users
	* specialized Nodes (GPU) to avoid running non-spec apps
	* taint by condition (alpha status feat). auto taint nodes with problems. allow to add tolerations to time eviction of pods
* to enable alpha feats. add `--feature-gates` to K8s controller manager. or in kops use kops edit to add
```
spec:
  kubelet:
    featureGates:
      TaintNodesByCondition: "true"
```
* taint / toleration useful keys
	* node.kubernetes.io/not-ready : node is not ready
	* node.kubernetes.io/unreachable : node unreachable from node controller
	* node.kubernetes.io/out-of-disk : node runs out of disk
	* node.kubernetes.io/memory-pressure : node has low memory
	* node.kubernetes.io/disk-pessure : node has disk pressure
	* node.kubernetes.io/network-unavailable : node network is unavailable
	* node.kubernetes.io/unschedulable : node is unschedulable

### Lecture 76 - Demo: Taints and Tolerations

* we run in vagrant vm on AWS cluster
* we run `kubectl get nodes <masternodename> -o yaml |less` and see the taint section where we see the rule for master added by k8s
* in kubernetes-course/tolerations we have README with instruction on taint nodes
* we taint a node `kubectl taint nodes ip-172-20-36-137.eu-central-1.compute.internal type=specialnode:NoSchedule`
* we deploy a Deployment tolerations.yml with tolerations in Pods. only second deploymnt with toleration is deployed in tainted node

### Lecture 77 - Custom Resource Definitions (CRDs)

* CRDs allow us to extend the Kubernetes API
* Resources are endpoints in Kubernetes API that store collections of API Objects
* Deployment is a built-in resource that we use to deploy applicaitons
* in YAML files we describe the object using the Deployment resource type
* the object is created on cluster with kubectl
* a Custom Resource is a resource we add to our cluster. its not available in every k8s cluster. it is a Extension to K8s API
* Custom Resources are also described in YAML files
* as an admin we can dynamically add CRDs to add extra functionality to the cluster
* Operators use these CRDs to extend the API with their own functionality

### Lecture 78 - Operators

* Operators is a method of packaging, deploying and managing a K8s app (CoreOS team)
* it puts operational knowledge to an application. it brings user closer to the experience of managed cloud services, rather than having to know the intricacis of app deployment on K8s
* Once Operator is deployed it is managed with CDRs
* It is a great way to deploy stateful services on K8s (hidding the complexity)
* Any 3rd party can create Operators. (Prometheus, Vault, Rook, MySQL,PostgreSQL)
* We can deploy PostgreSQL container.. this gives only the DB
* If we use the operator for PostgreSQL it allows to create replicas,initiate failove, create backups, scale.more like a Cloud Service
* operator contains management logic that an admin might want but does not want to write it from scratch
* with postgres operator we will use custom objects from other api versions

### Lecture 79 - Demo: Postgres operator

* postgres operator is available in [github](https://github.com/CrunchyData/postgres-operator)
* we are in vagrant vm with our cluster running
* we go to kubernetes-course/postgres-operator
* README contains all the commands
* first we apply storage.yml to create a StorageClass setting the correct zonevim 
* we install a ascript for GKE (works also for AWS) quickstart-for-gke.sh
* we select storage and deploy 
* we need to do a portforward to access the operator `kubectl port-forward  postgres-operator-5d4b49b4c8-6wtvt  18443:8443`
* to access the cli we add it to our PATH. we use a script for this `./set-path.sh` and logout and login to vm
* we check pgo installation with `pgo version` pgo client communicates with operator with port forwarding
* we create a cluster with pgo and see it
```
pgo create cluster  mycluster
pgo show cluster all
```
* what we get is
```
cluster : mycluster (centos7-10.4-1.8.3)
	pod : mycluster-7dff65997f-qbd4p (Pending) on  (0/0) (primary)
	pvc : mycluster
	deployment : mycluster
	service : mycluster (100.68.141.64)
	labels : primary=true archive=false archive-timeout=60 crunchy_collect=false name=mycluster pg-cluster=mycluster 
```
* with kubectl get pods we see that AWS cluster has 2 pods . cluster and pgo
* to view secrets from cluster `pgo show cluster mycluster --show-secrets=true`
* we connect to psql db with a psql image from dockerhub `kubectl run -it --rm --image=postgres:10.4 psql -- psql -h mycluster -U postgres -W` and login to our db
* we get nice feats with operator like the ability to scale our db with 1 command `pgo scale mycluster`
* we see our cluster wth `pgo show cluster mycluster` 
* we can create  a replica to another node while we maintain our node. we can apply strategies so sour cluster can survive node failure and availability zone failure
* we can use pgo to create failover for the cluster : manualy failover
```
pgo failover mycluster --query
pgo failover mycluster --target=mycluster-xvis

```
* when we query we get a replica as failover
* the second command does the failover
* pgtask created is a CRD we can see its definition with `kubectl get pgtasks mycluster-failover -o yaml`
* i can see the crds `kubectl get crd`

## Section 4 - Kubernetes Administration

### Lecture 80 - The Kubernetes Master Services

* Master Node Architecture Overview:
	* to communicate with our cluster we use kube control (kubectl) 
	* kubectl communicates to the REST API in the master node after it passes autorization. when we send new resources and create new obkects with kubectl using the REST API these resources are saved
	* kubernetes uses etcd (high availability cluster distributed datastore on 3 or 5 nodes) as backend where objects are stored
	* REST API (K8s API Server) communicates with etcd and Scheduler
	* Scheduler schedules pods not yet scheduled. it is pluggable. we can use any scheduler we want
	* Controller Manager: exists of multiple controllers e.g node controller, replication controller
	* REST interface communicates with kubelets in nodes

### Lecture 81 - Resource Quotas

* when a K8s cluster is used by multiple teams or people, resource management becomes very important
* we need to manage the resources we give to a person or a team. we dont want a paerson or team to take all cluster resources
* we can divide our cluster in namespaces and enable resource quotas on it
* we do this with ResourceQuota and ObjectQuota objects
* each container can specify 'request capacity' and 'capacity limits'
* Request capacity is an explicit request of resources
* Scheduler use the request capacity info to decide on where to put the pod on
* It represents the minimum amount of rsources the pod needs
* Resource limit is a limit imposed to the container. container cannot utilize more resources than specified
* Example of ResourceQuota:
	* we run a deployment with a pod with a resource of 200m (20% of single CPU core in node). a limit posed on pods could be 400m 
	* memory quotas are defined in MiB or GiB
* if a quota has been specified by the admin. each pod needs to specify capacity quota during creation
* admin can spec default request values for pods that dont spec any capacity vals
* same holds for limit quotas
* if a resource requests more that allowed capacity. k8s server API will give an error (403 forbidden) and kubectl will show error
* Resource limits in a namespace:
	* requests.cpu : the sum of CPU requests of all pods cannot exceed the value
	* requests.mem : the sum of MEM requests of all pods cannot exceed this val
	* requests.storage : the sum of storage requests of all persistent volume claims cannot exceed this val
	* limits.cpu : the sum of CPU limits of all pods cannot exceed this val
	* limits.memory the sum of MEM limits of all pods cannot exceed this val
* Admin set object limits:
	* configmaps : total num of configmaps that can exist in a namespace
	* persistentvolumeclaims
	* pods
	* replicationcontrollers
	* resourcequotas
	* services
	* services.loabalancer
	* services.nodeport
	* secrets

### Lecture 82 - Namespaces

* namespaces allow us to create virtual clusters within the same physical cluster
* namespaces logically separate your cluster
* standard namespace is 'default'. this is where resources are launched by default
* built-in namespace for k8s speciffic resources is 'kube-system'
* namespaces are handy when multiple teams or projects use the cluster
* resource name uniqueness req holds in namespaces not cluster
* quotas and limits can be set per namespaces
* create namespace `kubectl create namespace myspace`, get namespaces `kubectl get namespaces`
* to set default namespace (get default context set it)
```
export CONTEXT=$(kubectl config view| awk '/current-context/ {print $2}')
kubectl config set-context $CONTEXT --namespace=myspace
```
* to set limit on namespace we add `namespace: myspace` in ResourceQutoa metadata

### Lecture 83 - Demo: Namespace Quotas

* we work in minikube on host
* we cd in CourseRepo/resourcequotas
* in resourcequota.yml  we create a new namespace and define resource quotas in it
* we create the objects in YAML
* we deploy an app with no quotas (helloworld-no-quotas.yml)
* we see the sdepolyment status in namespasce with `kubectl get deploy --namespace=myspace` no pod is launched (no quota while there is request) failed quota
* when we deploy pods with quotas they are deployed. we hit the limit so only 2 are scheduled
* we can see quota usage with `kubectl get quota --namespace=myspace` and see the details with `kubectl describe quota/compute-quota --namespace=myspace`
* we can set default limits 'default.yml' then deploy without quotas. defaults are used instead of quotas

### Lecture 84 - User Management

* there are 2 types of users we can create
	* normal user. used to access the cluster externally e.g with kubectl. this user is not managed using objects
	* a Service user. managed by an object in K8s. this user is used to authenticate within the cluster. from inside a pod or from a kubelet. their credentials are managed by secrets
* Authentication strategies for normal users
	* Client Certificates
	* Bearer Tokens
	* Authentication Proxy
	* HTTP Basic Auth
	* OpenID
	* Webhooks
* Service Ysers use Service Authentication Tokens
	* they are stored as credentials using Secrets
	* these Secrets are mounted in pods to allow commbetween services
* Service Users are namespace specific
* they are created automatically by API or manually using objects
* any Non-authed API call is treated as anonymous user
* Norma users have following attributes
	* Username
	* UID
	* Groups
	* extrafields
* normal user can access everything. to limit access we need to config authorization
	* AlwaysAllow / AlwaysDeny
	* ABAC (Attribute Based Access Control)
	* RBAC (role based access control)
	* Webhooks
* RBAC is mostly used (uses rbac.authorization.k8s.io API group) as ABAC needs manual config
* RBAC allows dynamic permission config with the api

### Lecture 85 - Demo: Adding Users

* we use minikube. we ssh into the cluster `minikube ssh`
* we creare a new key using openssl `openssl genrsa -out sakis.pem 2048`
* we create a new certificate request specifying loging and group `openssl req -new -key sakis.pem -out sakis-csr.pem -subj "CN=sakis/O=myteam/"`
* we use minikube ca certificate and key to create a new cerificate signed by us `sudo  openssl x509 -req -in sakis-csr.pem -CA /var/lib/localkube/certs/ca.crt -CAkey /var/lib/localkube/certs/ca.key -CAcreateserial -out sakis.crt -days 10000`
* in that way we create a certificate we can use to authenticate to the minikube cluster API server
* we look in the certificate `cat sakis.crt` and `cat sakis.pem` and cp the key content to use it on host system
* on host we vim in `vim ~/.kube/config` and change apiserver.crt and key to sakis.crt akd key. 
* we also need to cp the content of keys in `vim ~/.minikube/sakis.key` and `vim ~/.minikube/sakis.crt` 
* we run `kubectl config view` anb see that we are using the new files to connect
* in corporate env  we use LDAP or activex

### Lecture 86 - RBAC

* after authentication authorization controls what a user can do and his access rights
* access controls are implemented on an API level (kube-apiserver)
* when API request comes (e.g kubectl get nodes) it will be checked to see whether w ehave access to execute the command
* The various authrization modules are:
	* Node: a special purpose authorization mode that authorizes API requests made by kubelets
	* ABAC: attribute based acceess control. rights controlled by policies that combine attributes
	* RBAC: rolebased access control. dynamic permission policies
	* Webhook: sends authorization req to external REST APi. good if we want to use our own external server. we can parse incoming JSON and reply with access deenied or granted
* to enable authorization mode. we need to pass --authorization-mode= to the API server at startup (e.g --authorization-mode=RBAC)
* RBAC is enabled by default in most tools (like kops and kubeadm) not in minikube (yet)
* if we use minikube we can pass a poara at startup `minikube start --extra-config=apiserver.Authorization.Mode=RBAC`
* we can add RBAC resources with kubectl to grant permissions. describe them in YAML then apply to cluster
* first we define a role and then add users/group to the role
* we can create roles limited to namespace or create roles with access aplied to all namespace. Role(namespace) ClusterRole(cluster-wide) RoleBinding (namespace) ClusterRoleBinding(cluster-wide) 
* in Role YAMl when we leave something empty it means all e.g apiGroup= [""]

### Lecture 87 - Demo:RBAC

* we are in vagrant vm on the AWS cluster running
* we go to kubernetes-course/users/ and see the READM on how to create a user
* to create a new user we need to create cerificate. and to do that we need our clusters certificate as issue authority
* we sync s3 key on aws with the local dir `aws s3 sync s3://kops-state-4213432/k8s.agileng.io/pki/private/ca/ ca-key`
* we sync s3 cert on aws with the local dir `aws s3 sync s3://kops-state-4213432/k8s.agileng.io/pki/issued/ca/ ca-crt`
* we move both key and crt into local dir renaming them `mv ca-key/*.key ca.key` `mv ca-crt/*.crt ca.crt
` 
* to create a new cert we need openssl
* we create a new key `openssl genrsa -out sakis.pem 2048
`
* using the key we make a certificate request `openssl req -new -key sakis.pem -out sakis-csr.pem -subj "/CN=sakis/O=myteam/"`
* we sign it using the ca.crt and ca.key `openssl x509 -req -in sakis-csr.pem -CA ca.crt -CAkey ca.key -CAcreateserial -out sakis.crt -days 10000`
* we now have user cert and key to login to the cluster. we create a new context adding entries in the cluster config `kubectl config set-credentials sakis --client-certificate=sakis.crt --client-key=sakis.pem`
* with user created we create a new context `kubectl config set-context sakis --cluster=k8s.agileng.io --user sakis`
* we verify the changes with `kubectl config view` we see the user
* if i `kubectl config get-contexts` i see the new context. we see that defualt context is current
* if i set my new context as current `kubectl config use-context sakis`
* now if i issue commands i get error (forbidden)
* i switch back to default user `kubectl config use-context k8s.agileng.io`
* the rolebindind for the new user definition is in admin-user.yml it is a ClusterRoleBinding tha binds sakis to the built-in role cluster-admin
* we apply it. switch context and now e can execute commands
* we switch back and delete the admin role. we apply user.yaml woith a stripped down role. switch backl context and test commands

### Lecture 88 - Networking

* Networking topics covered so far:
	* Container to container comm in a pod (localhost:port)
	* Pod-to-service comm (nodeport w/ DNS, ClusterIP)
	* External-to-service (LoadBalancer, Ingress NodePort)
* In k8s a pod should always be routable (Pod-to-pod comm)
* Kubernetes assumes pods should be able to comm to other pods, regardless of which node they are running
* Every pod has its own IP
* Pods on different nodes need to communicate to eachother using these IP addresses.
* the implemntation of interpod comm accross nodes is different per installation
* On AWS: kubenet networking (kops default). every pod gets an Ip which is routable using AWS VPC (virtual private network)
* the k8s master allocates /24 subnet to each node (254 IP addresses) 
* the subnet is added to the VPC route table
* there is a limit of 50 entries which means we cannot have more than 50 nodes in a single AWS cluster
* not all cloud providers have VPC technology (GCE, Azure have it)
* An available alternative is Container Network Interface (CNI) a SW that provides libs/plugins for network interfaces for containers. Popular solutions are Calico, Weave (standalone or w/ CNI)
* An other solution is using an Overlay Network: Flannel is an easy and popular way
* Flannel acts as a network gateway between nodes. it runs in the node as a service. in the node services have their IPs (virtual network) like 10.3.1.X (node1 vlan) and 10.3.2.X (node2 vlan) nodes have their own IPs in the cluster network (managed by cloud provider) like 192.168.0.2 and 192.168.0.3. packets have the nodes IPs to travel in the host network. but flannel injects a UDP payload encaptulating the node vlan ip addresses. so that pods can comm accross nodes. iptables are stored in etcd.

### Lecture 89 - Node Maintenance

* Node Controller is responsible for managing the Node objects
	* it assigns IP space to the nod ewhen a new node is launched
	* it keeps the nod elist up to date with available machines
	* the node controller monitors the health of the node: if a node gets unhealthy it gets deleted. pods running on an unhealthy node get rescheduled
* When adding a new node , kubelet will try to register itself
* it is called self-registration and is a default behaviour
* it allows to easily add more nodes to the cluster without making API changes
* the new node object is automatically created with: metadata (name,IP,hostname) and labels (cloud region, availability, instance)
* A node has a node condition (e.g Ready, OutOfDisk)
* To decommission a node (gracefully) we drain it (remove pods) before shuting down `kubectl drain <nodename> --grace-period=600`. If the nod eruns pods not managed by a controller `kubectl drain <nodename> --force`

### Lecture 90 - Demo: Node Maintenance

* we ll see how to drain a node in minikube
* we create a deployment CourseRepo/deployment/helloworld.yml
* all pods are in the single node. we drain it `kubectl drain minikube` we need to use --force

### Lecture 91 - High Availability

* if we want to run the cluster in production , we will want to have all our master services in a High Availability (HA) Setup
	* Clustering etcd: al least run 3 etcd nodes
	* Replicated API servers with a LoadBalancer
	* Running multiple instances of the scheduler  and the controllers (one leader , others in standby)
* etcd: 1 node (No HA) 3nodes (HA) 5nodes (Big cluster HA)
* With HA we have at least 2 master nodes.
	* load balancer directs traffic from client (kubectl) and nodes to the 2 masters
	* serices running on both: authorization, APIs (REST, scheduling actuator), kubectl and monit
	* services on standby: scheduler and controller manager
* kops hccan do the heavy lifting for production cluster on AWS. on other provider s we use other kube deployment tools
* kubeadm can set up a cluster for us [HA with no tooling](https://kubernetes.io/docs/setup/independent/high-availability/)

### Lecture 92 - Demo: High Availability

* in vagrant vm we create a new cluster addind master zones `kops create cluster --name=k8s.agileng.io --state=s3://kops-state-4213432 --zones=eu-central-1a, eu-central-1b,eu-central-1c --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8s.agileng.io --master-zones=eu-central-1a,eu-central-1b,eu-central-1c`
* we can spec multiple zones for nodes. 
* we spec multiple zones for master. 3 zones means 3 masters one in each zone
* we create the cluster and see the configuration `kops edit ig --name=k8s.agileng.io nodes --state=s3://kops-state-4213432` ig stands for instance groups
* we see the instancegroup config. if we want to move to 1zone we can remove 2 extra zones
* to see the master node we `kops edit ig --name=k8s.agileng.io master-eu-central-1a --state=s3://kops-state-4213432`
* we do the same inspecion to outher masters on other zones. if a zone fails we still have a master to run the cluster

### Lecture 93 - TLS on ELB using Annotations

* we can setup cloud specific feats (like TLS termination to use https) on AWS LoadBalancers that we create in K8s using services of type LoadBalancer
* we can do this using annotations: in the Service YAAML descript
* The possible annothations for the AWS ELB:
	* service.beta.kubernetes.io/aws-load-balancer-access-log-enit-interval, service.beta.kubernetes.io/aws-load-balancer-access-log-enabled, service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name, service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix are all used to enable access logs on the load balancer. we need also permissions
	* service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags (add tags)
	* service.beta.kubernetes.io/aws-load-balancer-backend-protocol (spec the backend protocol to use, the protocol that pod uses e.g http or https). https is of not use as we are in the cluster
	* service.beta.kubernetes.io/aws-load-balancer-ssl-cert (Certificate ARN)
	* service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled (connection draining)
	* service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout : client timeout when backend node stops during scaling
	* service.beta.kubernetes.io/aws-load-balancer-connection-idle0timeout
	* service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled CrossAZ balancing
	* service.beta.kubernetes.io/aws-load-balancer-extra-security-groups
	* service.beta.kubernetes.io/aws-load-balancer-internal set elb to internal lb mode (no public ip)
	* service.beta.kubernetes.io/aws-load-balancer-proxy-protocol Enable proxy protocol
	* service.beta.kubernetes.io/aws-load-balancer-ssl-ports : what listeners to enable HTTPS on (default is to all usually set to 443)

### Lecture 94 - Demo: TLS on ELB

* to terminate TLS on ELB we need to create an SSL certificate
* we go to AWS certificate manager. to create a cert
* check region => provision certificate => request a public cert => use our domain name  adding a subdomain 'helloworld.k8s.agileng.io' => next => dns validation => review => confirm and request => we need to add the CNAME record to our den provider as our domain is on route 53 we click 'Create record in Route53' => continue
* it is pending validation. we go to Route53 =>  hosted zone we see it added
* after some time it is issued. in its details we see and cp the ARN to use in YAML file
* we create a singlem master cluster
* in kubernetes-course/elb-tls we have 2 yaml files. 1 for deployment (3 pods) and one for the Service (loadbalancer) there we use annotations
```
service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:region:accountid:certificate/..." #replace this value
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
    service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=dev,app=helloworld
```
* we add the arn, backedn prot is http
* we use 2 ports in load balancer 80 and 443
* we apply it
* we get external ip with `kubectl get services -o wide` we add the hostname to route53 add a record and use the alias of elb hostname
* in EC2 => LoadBalancers => details i see 2 instances of ELB (2 nodes) listning to 2 ports
* both http and https works

## Section 5 - Packaging and Deploying on Kubernetes

### Lecture 95 - Introduction to Helm

* Helm: the best way to find, share and use SW built for Kubernetes
* a package manager for K8s. it helps manage K8s apps
* HELM is maintained by CNCF (Cloud Native Computing Foundation) like Kubernetes. Also by Google, MS and other
* to start using helm we need to download helm client
* we need to run 'helm init' to initialize helm on the K8s cluster
	* it will install Tiller
	* if we have RBAC (recent clusters have it by default) we need to add ServiceAccount and RBAC rules so that Helm can install Tiller
* After this helm is ready and we can start installing charts
* Charts is a packaging format used by Helm. it is a collection of files that describe a set of Kubernetes resources (like YAML files)
* A single chart can deploy an app. a piece of SW or a DB
* It can have dependencies
* We can write our own chart to deploy our app on Kubernetes using Helm
* Charts use templates that are developed by a package maintainer. they generate YAML files that K8s can understand. Templates are like dynamic YAML files, with logic and vars
```
apiversion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favoriteDrink }}
```
* in this template example. the 2 vars used can be overriden by the user when running helm install
* Common Helm Commands
	* helm init : install tiller on cluster
	* helm reset : Remove tiller from cluster
	* helm install : Install a helm chart
	* helm search : search for a chart
	* helm list : list releases (installed charts)
	* helm upgrade : upgrade release
	* helm rollback : rollback to a previous release