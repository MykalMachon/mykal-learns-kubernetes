## Mastering `kubectl`
This was a simple section. It went over things we've already learned in previous courses and didn't teach anything that we haven't already covered either in previous courses or our own research elsewhere.
## Running Kubernetes 
We went over three primary options for getting up and running with Kubernetes clusters
- **minikube**: single host k8s for testing and automation on a single host
- **kubeadm**: tool to send up bare metal k8s systems relatively simply given all nodes are on the same network
- kops: a cli tool to make it easy to create a cluster on GCP, Azure, or AWS. Specifically, the `kops create` and `kops update cluster` commands were covered. `kops validate cluster` is also useful for validating clusters  
- GCP KS/Azure KS/AWS EKS: 
	- if on AWS, you want to use KOPS for everything cause it's similar to kubectl and the aws cli and makes things very easy. AWS may have better tools now.

## Other useful tools 

### Kubernetes dashboard 
- can be used to visualize clusters and resources
- manage apps, troubleshoot issues, and manage the cluster via a UI 
- this can be installed by an add on in minikube or on a real cluster you can install it with helmcharts or manifest files.
- visiting the UI can be done via the `kubectl proxy` command and visit at localhost:8081/ui/ 
- cluster overview screen is a good way to check the health of all nodes quickly 
- k8s overview page gives an overview of the resources available and used
- the pod page is useful for editing and managing pod manifests and yaml information. 
- adds some devops tools. 
- there is also a login page system that is available now for access control and stuff

### Federation and `kubefed`
This makes it easy to manage multiple clusters by syncing resources across clusters in different regions to make sure that services are close to your users (i.e. cluster in NA, cluster in EU, cluster in Asia, etc.) In a hybrid cloud scenario, this also prevents lock-in.

`kubefed` is the key to setting up federation. You can initialize `kubefed` to create a control plane that syncs all your k8s clusters. You can add existing clusters to `kubefed` by just passing the config into `kubefed`. This is a deeply complicated system and this course is only making us aware of it not teaching us how to set it up or actually utilize it. 

### Kompose 
converts docker compose files into Kubernetes manifest files. This is meant to be an escape hatch from docker compose into k8s. This might be interesting to dig into for moving a lot of our existing infrastructure and tools into Kubernetes if needed at some point. 

all you need to do is 
- download the binary 
- place the binary in your path, use the kompose convert command to convert each compose to a manifest 
- review and apply your manifest into your cluster locally to test things. 

Not all docker compose keys are easily converted to k8s, so you should definitely make sure to manually review everything. 
### Helm
"think of this like a package manager for k8s". We already know about helm and how it all works thanks to the previous course so I'll keep notes light here. basically it allows for you to version your apps, provide rollback via tiller, and you can get started with k8s apps very easily thanks to open-source helmcharts.
## Next steps
- checkout the k8s docs
- checkout "Kubernetes the hard way" by Kelsey Hightower (second time I've heard this)
	- I've followed Kelsey on twitter for years and didn't realize he was a big k8s person. 
	- Good to know!
