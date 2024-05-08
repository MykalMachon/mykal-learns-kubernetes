.PHONY: run_website stop_website install_kind install_kubectl \ 
	create_kind_cluster

run_website: 
	docker build -t cheeselore.com:1.0 . && \
		docker run --rm --name cheeselore.com -p 8080:80 -d cheeselore.com:1.0

stop_website: 
	docker stop cheeselore.com

install_kubectl:
	curl --location --output ./kubectl "https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl" && \
		chmod +x kubectl && \
		sudo mv kubectl /usr/local/bin/kubectl && \
		kubectl version --client

install_kind: 
	curl --location --output ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.22.0/kind-linux-amd64 && \
		chmod +x ./kind && \
		sudo mv ./kind /usr/local/bin/kind && \
		kind --version

create_kind_cluster: install_kind install_kubectl
	kind create cluster --name cheeselore.com && \
		kubectl get nodes --namespace kube-system