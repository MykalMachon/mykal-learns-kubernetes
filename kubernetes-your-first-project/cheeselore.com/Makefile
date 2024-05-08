.PHONY: run_website stop_website install_kind install_kubectl create_cluster

run_website: 
	docker build -t cheeselore.com:1.0 . && \
		docker run --rm --name cheeselore.com -p 8080:80 -d cheeselore.com:1.0

stop_website: 
	docker stop cheeselore.com

install_kubectl:
	curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
		chmod +x kubectl && \
		mv kubectl /usr/local/bin/kubectl && \
		kubectl version --client

install_kind: 
	curl --location --output ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.22.0/kind-linux-amd64 && \
		chmod +x ./kind && \
		./kind --version

create_cluster: install_kind
	./kind create cluster --name cheeselore.com