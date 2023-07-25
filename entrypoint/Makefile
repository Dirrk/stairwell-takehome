
ecr-repository ?= 359294263934.dkr.ecr.us-west-2.amazonaws.com
docker-image ?= app/stairwell-entrypoint
docker-image-tag ?= latest

ecr-login:
	aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin ${ecr-repository}

docker-build: ecr-login
	docker build -t ${ecr-repository}/${docker-image}:${docker-image-tag} .

docker-push: docker-build
	docker push ${ecr-repository}/${docker-image}:${docker-image-tag}

deploy:
	kubectl apply -f entrypoint.yaml
