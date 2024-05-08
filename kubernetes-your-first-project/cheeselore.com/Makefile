.PHONY: run_website stop_website install_kind install_kubectl \ 
	create_kind_cluster create_docker_registry

run_website: 
	docker build -t cheeselore.com:1.0 . && \
		docker run --rm --name cheeselore.com -p 8080:80 -d cheeselore.com:1.0

stop_website: 
	docker stop cheeselore.com

install_kubectl:
	if kubectl version --client | grep -q "Client Version"; \
	then echo "---> kubectl already installed; skipping"; \
	else curl --location --output ./kubectl "https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl" && \
		chmod +x kubectl && \
		sudo mv kubectl /usr/local/bin/kubectl; \
	fi && \
	kubectl version --client

install_kind: 
	if kind version | grep -q "kind"; \
	then echo "---> kind already installed; skipping"; \
	else curl --location --output ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.22.0/kind-linux-amd64 && \
		chmod +x ./kind && \
		sudo mv ./kind /usr/local/bin/kind; \
	fi && \
	kind --version

create_docker_registry:
	if docker ps | grep -q 'local-registry'; \
	then echo "---> local-registry already exists; skipping"; \
	else docker run -d -p 5000:5000 --restart=always --name local-registry registry:2; \
	fi && \
	curl -L http://localhost:5000/v2

create_kind_cluster: install_kind install_kubectl create_docker_registry
	kind create cluster --name cheeselore.com && \
		kubectl get nodes --namespace kube-system

