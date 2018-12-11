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
* we can use kops to create a cluster on aws with 1 master and 2 nodes `kops create cluster --name=k8s.agileng.io --state=s3://kops-state-4213432 --zones=eu-central-1 --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8s.agileng.io`
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
* we expose this deployment in the cluster using the dev service NodePort `kubectl expose deployment hello-minicube --type=NodePort`
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