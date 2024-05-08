.PHONY: run_website stop_website install_kind

run_website: 
	docker build -t cheeselore.com:1.0 . && \
		docker run --rm --name cheeselore.com -p 8080:80 -d cheeselore.com:1.0

stop_website: 
	docker stop cheeselore.com

install_kind: 
	echo "todo: install kind"