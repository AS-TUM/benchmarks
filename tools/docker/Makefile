# Get full path to this Makefile's directory
DOCKERHUB_DIR := $(dir $(realpath $(lastword $(MAKEFILE_LIST))))
# The Dockerfile is in: ThermSniper/external/benchmarks/tools/docker/
# So go 5 levels up to get the full ThermSniper_with_benchmarksupport/ project root
PROJECT_ROOT := $(abspath $(DOCKERHUB_DIR)/../../../../..)

DOCKERLOCAL_IMAGE=benchmarks-$(USER):latest
DOCKERHUB_BUILDER_IMAGE=snipersim/benchmarks-builder:latest
DOCKERHUB_IMAGE=snipersim/benchmarks:latest
DOCKERHUB_SNIPER_IMAGE=snipersim/snipersim:latest
DOCKERBASE_IMAGE=benchmarks-basebuilder-$(USER):latest
DOCKERPASSTHROUGH_IMAGE=benchmarks-passthrough-$(USER):latest

SNIPER_GIT_REPO?=https://github.com/snipersim/snipersim
export SNIPER_ROOT=$(PROJECT_ROOT)/sniper

# Local build of Sniper based on the Docker Hub base image
run: build
	docker run --rm -it $(DOCKERLOCAL_IMAGE)

build: Dockerfile
ifndef SNIPER_GIT_REPO
	$(error Please set the SNIPER_GIT_REPO variable)
endif
	docker build \
		-f $(DOCKERHUB_DIR)/Dockerfile \
		--target dockerhub \
		--build-arg SNIPER_GIT_REPO=$(SNIPER_GIT_REPO) \
		-t $(DOCKERLOCAL_IMAGE) \
		$(PROJECT_ROOT)

# Docker Hub targets unchanged
runhub: buildhub
	docker run --rm -it $(DOCKERHUB_IMAGE)

# push_benchmarks: buildhub
# 	docker push $(DOCKERHUB_BUILDER_IMAGE)
# 	docker push $(DOCKERHUB_IMAGE)

# push_sniper: build
# 	docker rmi --no-prune $(DOCKERHUB_SNIPER_IMAGE) || true
# 	docker tag $(DOCKERLOCAL_IMAGE) $(DOCKERHUB_SNIPER_IMAGE)
# 	docker push $(DOCKERHUB_SNIPER_IMAGE)

# push: push_benchmarks push_sniper

# buildhub: Dockerfile.dockerhub
# 	docker build -f ./Dockerfile.dockerhub --target builder -t $(DOCKERHUB_BUILDER_IMAGE) .
# 	docker build -f ./Dockerfile.dockerhub --target runner -t $(DOCKERHUB_IMAGE) .

# # Build recipes for passthrough images (using local directories to store the benchmark and Sniper data)
# runpassthrough: buildpassthrough
# 	docker run --rm -it -e HOME="${HOME}" -v "${HOME}:${HOME}" --user $(shell id -u):$(shell id -g) -w "${PWD}" $(DOCKERPASSTHROUGH_IMAGE)

# buildpassthrough: Dockerfile.passthrough
# 	docker build -f ./Dockerfile.dockerhub --target basebuilder -t $(DOCKERBASE_IMAGE) .
# 	docker build -f ./Dockerfile.passthrough --target builderpassthrough --build-arg FROMLOCATION=$(DOCKERBASE_IMAGE) --build-arg UID=$(shell id -u) --build-arg GID=$(shell id -g) --build-arg USER="${USER}" --build-arg GROUP=$(shell id --group --name) -t $(DOCKERPASSTHROUGH_IMAGE) .

.PHONY: run build runhub push push_benchmarks push_sniper buildhub runpassthrough buildpassthrough
