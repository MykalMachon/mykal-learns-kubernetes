# 1.0 Kubernetes and the cloud native ecosystem

## What is Kubernetes?

also called k8s or "kates". This is a container orchestration software that is written in [[Golang]]

### History

- Released by google in 2014
- Worked with the open Linux foundation and then donated Kubernetes to the CNCF ([[Cloud Native Computing Foundation]])

### Other

- Handles the scaling up/down
- Handles errors and crashes and self-healing

### Examples of companies

- Spotify has experienced explosive growth. in 2018 they migrated from an in-house solution to Kubernetes
- The biggest surface running on Kubernetes takes 10 million RPS and will automatically scale up and down saving money on compute.
- Before it would take hours to deploy now it takes seconds or minutes

### Where can it run

- anywhere
- Can run BareMetal
- All cloud services usually provide Kubernetes as a service.

## What are containers?

let's review, popularized by [[Docker]] they're a tech that bundles config, deps, and code to run an application in one unit

### Advantages

- portability: can run on any OS as long as there is a container agent
- efficient: less cpu / ram /resources than a VM
- self-container: can be spun up or down in seconds
- quick replications to automatically scale up / down as needed

### Terms

- Image: a container image with a file with executable code that can be run as a container
- Container Registry: a database of images; these images can be available to the public or private for those people with service accounts or permissions
 	- Docker Hub
 	- Azure Container Registry
 	- Google Container Registry
 	- Quay

### What we will be doing

**In this course we'll be using prebuilt images, if you want to learn how to build these do a different course**

- Telling Kubernetes what image we want from a registry
- Telling Kubernetes how many replicas we want to run
- Use the container runtime interface to build and start he containers

### Alternatives to docker

- podman
- rexd
- rocket

## What is cloud native technology?

This is a loaded term. It means a whole bunch of things and everyone will have a diffferent definition. Here's the official definition

> "Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach."
>
> - [CNCF.io](https://cncf.io)

here's the profs definition

> Cloud-native technologies are open-source projects designed to let technologists use cloud computing services to automatically deploy and scale applications.

It is good to remove silos between system admins, developers, and qa/other tech contributors. the CNCF's goal is to make this work and make this approach ubiquitous. The CNCF also makes a number of certificates for cloud native projects.

They also keep track of projects by grading them by maturity
![[Pasted image 20240417090816.png]]

### CNCF Open Source Projects

- some other examples are
 	- helm
 	- prometheus
 	- containerd

# 2.0 Setting up and getting oriented

We simply installed docker on linux (already done!) and then setup [minikube](https://minikube.sigs.k8s.io/docs/) on my WSL installation as well. From there we got into exploring a minikube cluster

## What is minikube

minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows. We proudly focus on helping application developers and new Kubernetes users. Getting this setup was really really easy. You can follow the instructions here <https://minikube.sigs.k8s.io/docs/start/>

## Difference between minikube and kubectl

minikube generates t he cluster, and kubectl lets you explore your cluster.

## Commands

- `kubectl cluster-info` returns cluster info URL
- `kubectl get nodes` returns the nodes in your cluster
- `kubectl get namespaces` returns the namespaces in your cluster. Namespaces are ways to isolate and manage applications and services
- `kubectl get pods -A` this shows all pods in every namespace. the `-A` flag means all in all namespaces. Pods are how containers run in kubernetes.
- `kubectl get services -A` returns all services currently running. services act as load balancers to the pods, and direct traffic to pods.

# 3.0 Application deployment

## Digging into YAML

The first thing we're gonna do is learn to write YAML to set up things through Infrastructure as code and gitops. This approach allows us to concretely define our configs in config files that are committed to a git repo, commits to that repo and merges into subsequent branches trigger actions in our cloud provider or generally in our Kubernetes cluster. You can learn more about gitops in a course called "gitops foundations" in LinkedIn learning.

**In Kubernetes, the most common config file is a YAML manifest.**

### What is YAML?

- data serialization file used for configuration and management between data systems.
- This makes data portable and save-able
- other examples of similar technology is JSON or TOML
- yaml stands for yaml aint markup language (recursion!)
- knowing how yaml syntax works should help with Kubernetes manifests

### YAML pointers

- a line that begins with `#` is a comment
- `---` denotes the start of a document. One file can have multiple documents
- everything is based on key values seperated by colons
- lists or sequeneces are denoted similar to markdown. The key is denoted as normal then you add some indented lists with dashes `-`
- can use either .yml or .yaml
- easy to mess up indentation, run things through a validator through <http://yamlchecker.com> or something similar

## Creating a namespace

A good example is creating a development and production namespace to properly segregate our namespaces. This allows you to isolate workloads so you can have dev/prod running on the same cluster.  

to get namespaces run `kubectl get namespaces`

### Looking at a namespace manifest

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

### adding the namespace

you can add the namespace file with the following `kubectl apply -f ./<your-file>.yml`
you can define multiple namespaces in one file because of the new document syntax

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
---
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

### deleting these namespaces

you can delete namespaces using `kubectl delete -f ./<your-file>.yml`

## Deploying an application

pods are the kubernetes resource that run our container resources. The best way to organize your pods to be highly available is using a **kubernetes deployment**.

Here's an example of a deployment

```yml
--- 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-info-deployment
  namespace: development
  labels:
    app: pod-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-info
  template:
    metadata:
      labels:
        app: pod-info
    spec:
      containers:
      - name: pod-info-container
        image: kimschles/pod-info-app:latest
        ports:
        - containerPort: 3000
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
```

To create the deployment, use the apply command on this file

### Ensuring this was created as expected

- `kubectl get deployments -n development` will show deployment specs that are currently installed
- `kubectl get pods -n development` will show pods that are currently ready to run.

### Deleting pod

you can delete a pod by grabbing it's name and running delete pod. You can then see the kubernetes cluster bring it back up.

## Checking the health of a pod using event logs

`kubectl describe pod <pod-name> -n development` your event logs are at the bottom and can help sort things out

## Check that your application is working with busybox

lets create a pod running busybox. We created a custom deployment for this as well.

- `kubectl get pods -n development -o wide` gets more information from your pods
- lets get into the pod so we can try and ping one of our pods from within our busy box commands. This is like docker exec -it
- `kubectl exec -it busybox-74b7c48b46-67fjz -- /bin/sh` will log ya into the container so you can run some stuff in it
- when running in this host, you should be able to get to the IP by pinging it OR using a wget command `wget <ip>:3000` you use port 3000 because that's what we defined previously.

### Reviewing

this is a good way to verify your application is working by getting a get request inside of your kubernetes environment and making sure it works.

## Viewing your application logs

- this is where I stopped for the day on [[2024-04-17]]
- get the pod name with `kubectl get pods`
- then run a `kubectl logs <pod-name> -n development`
- this will show the logs from each application to make sure it's working
- you can view logs in general for your apps here.

## Challenge: create your own deployment

- I did this; basically created our own deployment helm file and then tested it with busy box.
- was super easy to do
- I did this the right way given the following video that was the solution

# 4.0 Complex application deployment

## Exposing your application via a Load Balancer

You need to create a Kubernetes Service for this. You can create a Kubernetes Service of type load balancer that directs traffic from the internet to your pods.

This service has a public and static IP address. The public IP makes it accessible from the internet, it being static means that it is reliably accessible.

To do this, create a load balancer service and you have to run `minikube tunnel` to tunnel traffic from your minikube object out to the host machine.

## Add resource requests and limits to your pod

well configured containers let kubernetes know how much memory and CPU to reserve on a worker node for it. Resources refers to the amount of available CPU and memory on the worker running on your pods.

If you release a pod without resource requests, it can be scheduled on a node that doesn't have enough resource for it which will cause a node failure. Without resource limits, the node also won't know when to spin out to other nodes or can *take over* your whole node.

Here's a resource on resource management for containers: https:/
kubernetes.io/docs/concepts/configuration/manage-resources-containers/

here's an example

```
resources:
 requests:
     memory: "64mi"
        cpu: "250m"
    limits:
        memory: "128mi"
        cpu: "500m"
```

requests defines the bare minimum on the worker. The limits defines when to kill the container

## Delete your Kubernetes objects & tear down your cluster

Minikube is heavy, let's learn how to turn it off properly.

- the first thing to do is use the yaml files to delete all the Kubernetes resources we created
- this can be done by using the yaml files we built. just run `kubectl delete -f <file-path>`
- you can then delete the cluster by using `minikube delete`.
- I did not delete the cluster cause I will probably reuse it :)

# 5.0 Kubernetes architecture

## The Kubernetes control plane

starting with vocabulary

- each instance k8s is called a cluster
- each cluster has a control plane and at least 1 worker node
- the control plane is like an airport. it is made of several components

### API

Primary part of the control plane. it allows you to control your k8s instance over a REST based HTTP api. `kubectl` is simply a wrapper of this API that provides command line access.  To see the resources you have access to, you can run `kubectl api-resources`.

You should be able to see this running as a pod by running `kubectl get pods -n kube-system`.
Without this component a k8s cluster cannot exist

### etcd

only the [[#API]] server can communicate directly with this service. it's a highly available in-memory key-value store system and is used to manage api state. To view it's logs, you can run `kubectl logs etcd-minikube -n kube-system`.

### Scheduler

this looks for newly created pods that have not been assigned a worker node and chooses a node for the pod to run on. this is also run on a pod. it can be described or logs can be viewed

### Controller manager (cm)

runs as a loop to check that everything is working properly. It will check that all nodes are working, if they're broken, it will replace it, etc. It does a whole lot more than this but that's just an example.

### Cloud controller manager

lets you connect out to external services in the cloud so you don't have to have everything in your system.

> NOTE: if you're working in a managed Kubernetes instance like AKS (Azure Kubernetes Service) or AWS EKS they all of these nodes will be hidden from you as the cloud provider handles them.
>
## Kubernetes worker nodes

control plane is like the air traffic control tower, the nodes are like the terminals where planes show up and people board.

- Most k8s clusters run with at least 3 worker nodes to maintain high availability
- the worker nodes are where pods are scheduled and run.
- each worker node has three components
 	- **kubelet**: an agent that runs on every worker node and makes sure that containers are running and healthy. Communicates directly with the API server
 	- **container runtime**: after being assigned a new pod the kubelet starts a container using the container runtime interface (CRI). The cri enables the kubelet to create containers with the following engines: containerd, cri-o, kata containers, aws firecracker.
  		- they used to be able to use the docker runtime but it was removed in 1.24
 	- **kube proxy**: makes sure that pods and services can communicate on other pods and services on nodes and in the control plane. This also talks directly to the [[#API]] server.

## How the control plane and nodes work together

Gonna go over a time sequence diagram showing how this all works.
![[Pasted image 20240424094515.png]]

1. this all makes sense to be honest! pretty simple once you look at the time sequence diagram and see how all these services interact.
2. This is just one operation that your k8s cluster has to deal with. The API can often handle hundreds or thousands of requests to remain up and operational.
3. For more context on this see "beyond block diagrams: different ways to k8s architecture"

# 6.0 Advanced topics

## Ways to manage Kubernetes pods

- deployment is the most common way to deploy things.
 	- Allows control of replicas
 	- keeps old version up and running and rolls out new version, ensures health, then remove old pds.
 	- no downtime upgrades come for free with this model
- daemon set
 	- will put one copy of a container on every running worker node
 	- background processes typically run on dameonsets and collect analytics or logs for that node
- job
 	- will create one or more pods until the container has completed
 	- for example, imagine a container that runs to generate and seed testing data into your cluster
 	- or imagine if you want to run something on a set schedule

## Running stateful workloads

- how do you handle data storage in k8s?
- option 1: connect your app with a database running outside of your cluster. For example, imagine using postgresql
 	- either build and maintain it somewhere else
 	- or use something like azure sql, or some paas
- option 2: k8s persistent volumes
 	- these are data storage items that exist in your cluster and exist even after a pod is destroyed
 	- you can use a k8s object called a stateful set to make sure your updated application can use this data properly.
 	- This is a huge area and could have it's own course to be honest. Option 1 is much simpler.

## Kubernetes security

like any service on the internet, k8s is succeptable to attacks. Thankfully there are some best practices you can apply to help.

- hackers are often wanting to achieve 1 of 3 things
 	- steal data
 	- steal compute
 	- cause a DDoS
- add security context info to your pods. via the `securityContext` keys in your deployment yaml
 	- the first thing is to make sure you containers are run as a non-root user
 	- execing in to the pod gives a low level of access to your container.
 	- the second thing you can do is make your filesystem read only meaning that you cannot actually do anything in the filesystem.
 	- ![[Pasted image 20240424095637.png]]
- scanning your k8s manifest files using a CLI tool like Snyk (sneak)
 	- this will scan your manifest for possible holes and out of date containers
 	- you can do this with `snyk iac test ./<your-file>.yml`
 	- you can automate this in deploy using [[GitHub Actions]] or similar.
 	- it will show all the holes in your deployment that can be statically detected.
- you should also regularly update the version of Kubernetes you are using.
 	- check out the Kubernetes hardening guide released by the nsa
