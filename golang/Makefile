REGISTRY ?= ghcr.io
USERNAME ?= sergelogvinov
OCIREPO ?= $(REGISTRY)/$(USERNAME)
PLATFORM ?= linux/arm64,linux/amd64
PUSH ?= false

SHA ?= $(shell git describe --match=none --always --abbrev=7 --dirty)
TAG ?= $(shell git describe --tag --always --match v[0-9]\*)
GO_LDFLAGS := -ldflags "-w -s -X main.version=$(TAG) -X main.commit=$(SHA)"

OS ?= $(shell go env GOOS)
ARCH ?= $(shell go env GOARCH)
ARCHS = amd64 arm64

BUILD_ARGS := --platform=$(PLATFORM)
ifeq ($(PUSH),true)
BUILD_ARGS += --push=$(PUSH) --output type=image,annotation-index.org.opencontainers.image.source="https://github.com/$(USERNAME)/devops-examples"
else
BUILD_ARGS += --output type=docker
endif

############

# Help Menu

define HELP_MENU_HEADER
# Getting Started

To build this project, you must have the following installed:

- git
- make
- golang 1.23+
- golangci-lint

endef

export HELP_MENU_HEADER

help: ## This help menu
	@echo "$$HELP_MENU_HEADER"
	@grep -E '^[a-zA-Z0-9%_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

############
#
# Build Abstractions
#

build-all-archs:
	@for arch in $(ARCHS); do $(MAKE) ARCH=$${arch} build ; done

.PHONY: clean
clean: ## Clean
	rm -rf bin .cache

build-%:
	CGO_ENABLED=0 GOOS=$(OS) GOARCH=$(ARCH) go build $(GO_LDFLAGS) \
		-o bin/$*-$(ARCH) ./cmd/$*

.PHONY: build
build: build-my-service ## Build

.PHONY: lint
lint: ## Lint Code
	golangci-lint run --config .golangci.yml

.PHONY: unit
unit: ## Unit Tests
	go test -tags=unit $(shell go list ./...) $(TESTARGS)

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
images: image-my-service ## Build images
