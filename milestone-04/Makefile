# Search for .env File
ifneq (,$(wildcard ./.env))
    include .env
    export
endif

VENV_DIR = venv
PYTHON = $(VENV_DIR)/bin/python
PIP = $(VENV_DIR)/bin/pip
DOCKER_USER?=bloodraven21
DOCKER_IMAGE?=flask-api
TAG?=1.0.0


# Check if the virtual environment exists; if not, create it........
$(VENV_DIR)/bin/activate: requirements.txt
	@test -d $(VENV_DIR) || ( \
		echo "Creating virtual environment..."; \
		python3 -m venv $(VENV_DIR); \
		$(PIP) install --upgrade pip; \
		$(PIP) install -r requirements.txt \
	)

#  Run database using the docker compose.
run-db:
	docker compose up -d mysql && sleep 10

# Install dependencies inside the virtual environment.
install-dependencies: $(VENV_DIR)/bin/activate
	@echo "Installing dependencies..."
	$(PIP) install -r requirements.txt

#Target to initiate the migration.
init-migration:
	$(PYTHON) -m flask db init

# Target to generate the migration. (The message must comply with the app's requirements)....
generate-migration:
	$(PYTHON) -m flask db migrate -m "(Automatic generate the migration)"

# Run tests using pytest, ensuring the test database exists
test-api: $(VENV_DIR)/bin/activate
	DATABASE_URL=$(DATABASE_URL) python3 -m pytest

# Linting target
lint-api: 
	@echo "Running flake8 on app and tests directories..."
	python3 -m flake8 app tests

# Docker build the application...
docker-build:
	@echo "Building Docker image"
	docker build -t $(DOCKER_IMAGE):$(TAG) . 

# Tagging.
docker-tag:
	@echo "Tagging Docker image"
	docker tag $(DOCKER_IMAGE):$(TAG) $(DOCKER_USER)/$(DOCKER_IMAGE):$(TAG)

# Docker push the built image
docker-push: docker-build docker-tag
	@echo "Pushing Docker image to DockerHub"
	docker login -u $(DOCKER_USER) -p $(DOCKER_PASSWORD)
	@echo "Pushing Docker image to Docker Hub..."
	docker push $(DOCKER_USER)/$(DOCKER_IMAGE):$(TAG)

#  Run Flask-api using the docker compose
run-api:
	docker compose --env-file .env up -d

# Run the complete pipeline (build, tag, and push)
all:run-db install-dependencies init-migration generate-migration  test-api  docker-build docker-tag docker-push

# Target to stop container 
stop:
	docker compose down

# Cleanup: remove virtual environment and other generated files (but not migrations)
clean:
	rm -rf $(VENV_DIR)
	find . -name '*.pyc' -delete
	find . -name '__pycache__' -delete

# Full cleanup, including Alembic migrations (use with caution)
clean-full: clean
	rm -rvf .mypy_cache .pytest_cache/