# Search for .env File
ifneq (,$(wildcard ./.env))
    include .env
    export
endif

# Define version as a variable for easy updating
IMAGE_NAME = flask-api
DOCKERFILE_PATH = Dockerfile
TAG = 3.1.0


run:
	docker compose up

build-flask:
	docker build -t $(IMAGE_NAME):$(TAG) .

make stop:
	docker-compose down