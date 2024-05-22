.PHONY: run_website stop_website install_kind install_kubectl \ 
	create_kind_cluster create_docker_registry connect_registry_to_kind_network \
	connect_registry_to_kind create_kind_cluster_with_registry \ 
	delete_kind_cluster delete_registry delete_kind_cluster_with_regitry

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
	kind create cluster --name cheeselore.com --config ./kind_config.yml || true && \
		kubectl get nodes --namespace kube-system

connect_registry_to_kind_network: 
	docker network connect kind local-registry || true

connect_registry_to_kind: connect_registry_to_kind_network 
	kubectl apply -f ./kind_configmap.yml

create_kind_cluster_with_registry:
	$(MAKE) create_kind_cluster && $(MAKE) connect_registry_to_kind

delete_kind_cluster:
	kind delete cluster --name cheeselore.com

delete_registry:
	docker stop local-registry && docker rm local-registry 

delete_kind_cluster_with_regitry:
	$(MAKE) delete_kind_cluster && $(MAKE) delete_registry
	
push_image_to_registry:
	docker tag cheeselore.com localhost:5000/cheeselore.com && \
		docker push localhost:5000/cheeselore.com

deploy_website_to_kind: push_image_to_registry
	kubectl apply -f ./deployment.yml 

port_forward_website:
	kubectl port-forward service/cheeselore-svc 8080:80

delete_website_from_kind:
	kubectl delete -f ./deployment.yml
