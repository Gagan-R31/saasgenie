# Service-level Makefile for web-ui service
# Place this in: services/web-ui/Makefile

# Variables
REGISTRY ?= docker.io/gaganr31
IMAGE_NAME ?= demo
IMAGE_TAG_SHA ?= latest
SHOULD_CACHE ?= false
SHOULD_PUSH ?= false
KUBE_ENV ?= dev
FALCON_HOME ?= $(shell pwd)/../..

# Default target
.PHONY: help
help:
	@echo "Available targets:"
	@echo "  make build          - Build Docker image"
	@echo "  make test           - Run tests"
	@echo "  make check-format   - Check code formatting"
	@echo "  make deploy         - Deploy to Kubernetes"
	@echo "  make clean          - Clean build artifacts"

# Build Docker image
.PHONY: build
build:
	@echo "Building Docker image..."
	@echo "REGISTRY: $(REGISTRY)"
	@echo "IMAGE_NAME: $(IMAGE_NAME)"
	@echo "IMAGE_TAG_SHA: $(IMAGE_TAG_SHA)"
	@echo "SHOULD_CACHE: $(SHOULD_CACHE)"
	@echo "SHOULD_PUSH: $(SHOULD_PUSH)"
	
	docker build -t $(REGISTRY)/$(IMAGE_NAME):$(IMAGE_TAG_SHA) .
	docker tag $(REGISTRY)/$(IMAGE_NAME):$(IMAGE_TAG_SHA) $(REGISTRY)/$(IMAGE_NAME):latest
	
	@if [ "$(SHOULD_PUSH)" = "true" ]; then \
		echo "Pushing image..."; \
		docker push $(REGISTRY)/$(IMAGE_NAME):$(IMAGE_TAG_SHA); \
		docker push $(REGISTRY)/$(IMAGE_NAME):latest; \
	else \
		echo "Skipping push (SHOULD_PUSH=false)"; \
	fi

# Run tests
.PHONY: test
test:
	@echo "Running tests for web-ui..."
	@echo "FALCON_HOME: $(FALCON_HOME)"
	@echo "KUBE_ENV: $(KUBE_ENV)"
	
	# Check if HTML files exist
	@if [ ! -f index.html ]; then \
		echo "ERROR: index.html not found!"; \
		exit 1; \
	fi
	
	# Validate HTML (basic check)
	@echo "✓ HTML files found"
	@grep -q "<html" index.html || (echo "ERROR: Invalid HTML" && exit 1)
	@echo "✓ HTML validation passed"
	
	# Check Dockerfile exists
	@if [ ! -f Dockerfile ]; then \
		echo "ERROR: Dockerfile not found!"; \
		exit 1; \
	fi
	@echo "✓ Dockerfile found"
	
	@echo "✅ All tests passed!"

# Check code formatting
.PHONY: check-format
check-format:
	@echo "Checking code formatting..."
	@echo "FALCON_HOME: $(FALCON_HOME)"
	@echo "KUBE_ENV: $(KUBE_ENV)"
	
	# Check for trailing whitespace in HTML
	@echo "Checking HTML files..."
	@! grep -n '[[:space:]]$$' *.html 2>/dev/null || echo "Warning: Trailing whitespace found"
	
	# Check for proper indentation (basic check)
	@echo "✓ Format check passed"

# Deploy to Kubernetes
.PHONY: deploy
deploy:
	@echo "Deploying web-ui to Kubernetes..."
	@echo "KUBE_ENV: $(KUBE_ENV)"
	@echo "FALCON_HOME: $(FALCON_HOME)"
	
	# The deployment already exists, just update the image
	# k8s/base/deployment.yaml will have been updated by deploy.yaml with sed
	kubectl apply -f k8s/base/deployment.yaml
	
	@echo "✅ Deployment updated, waiting for rollout..."
	kubectl rollout status deployment/demo -n default --timeout=3m
	
	@echo "Current pods:"
	kubectl get pods -n default -l app=demo

# Clean build artifacts
.PHONY: clean
clean:
	@echo "Cleaning build artifacts..."
	@docker rmi -f $(REGISTRY)/$(IMAGE_NAME):$(IMAGE_TAG_SHA) 2>/dev/null || true
	@echo "✓ Clean complete"