#

REGISTRY ?= ghcr.io
USERNAME ?= sergelogvinov
OCIREPO ?= $(REGISTRY)/$(USERNAME)
PLATFORM ?= linux/arm64,linux/amd64
PUSH ?= false

SHA ?= $(shell git describe --match=none --always --abbrev=7 --dirty)
TAG ?= $(shell git describe --tag --always --match v[0-9]\*)

BUILD_ARGS := --platform=$(PLATFORM)
ifeq ($(PUSH),true)
BUILD_ARGS += --push=$(PUSH) --output type=image,annotation-index.org.opencontainers.image.source="https://github.com/$(USERNAME)/devops-examples"
else
BUILD_ARGS += --output type=docker
endif

THIS_FILE := $(lastword $(MAKEFILE_LIST))

############

# Help Menu

define HELP_MENU_HEADER
# Getting Started

To build this project locally, you must have the following installed:

- git
- make
- python
- docker
- docker-compose

endef

export HELP_MENU_HEADER

help: ## This help menu
	@echo "$$HELP_MENU_HEADER"
	@grep -E '^[a-zA-Z0-9%_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

############
#
# Build Abstractions
#

.PHONY: clean
clean: ## Clean
	rm -rf bin .cache

.PHONY: build
build:
	@docker compose -f docker-compose.yml build

.PHONY: lint
lint: ## Lint Code
	@echo "not implemented"

.PHONY: unit-stop
unit-stop:
	@docker compose -f docker-compose.yml down ||:

.PHONY: unit
unit: ## Unit Tests
	@echo "not implemented"

############
#
# Docker Abstractions
#

.PHONY: docker-init
docker-init:
	docker run --rm --privileged multiarch/qemu-user-static:register --reset

	docker context create multiarch ||:
	docker buildx create --name multiarch --driver docker-container --use ||:
	docker context use multiarch
	docker buildx inspect --bootstrap multiarch

image-%:
	docker buildx build $(BUILD_ARGS) \
		--build-arg TAG=$(TAG) \
		--build-arg SHA=$(SHA) \
		-t $(OCIREPO)/$*:$(TAG) \
		--target $* \
		-f Dockerfile .

.PHONY: images
images: image-release ## Build images
