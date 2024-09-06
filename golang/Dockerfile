# syntax = docker/dockerfile:1.8
########################################

# Use build architecture, golang can build for all architectures natively.
FROM --platform=${BUILDPLATFORM} golang:1.23-alpine3.20 AS builder

# Install basic tools
RUN apk update && apk add --no-cache make git
ENV GO111MODULE=on
WORKDIR /src

# Since go.mod doesn't change frequently, we can reuse the cache.
COPY ["go.mod", "/src"]
RUN go mod download && go mod verify

# Copy the source code without files defined in dockerignore
COPY . .

# ARGs always change, so they should be placed at the end of the Dockerfile.
ARG TAG
ARG SHA
# Build the binary for all architectures, see the Makefile for details.
RUN make build-all-archs

########################################

FROM --platform=${TARGETARCH} scratch AS my-service
LABEL org.opencontainers.image.source="https://github.com/sergelogvinov/devops-examples" \
      org.opencontainers.image.licenses="MIT"

# To make one layer
COPY --from=gcr.io/distroless/static-debian12:nonroot . .

# TARGETARCH is predefined by the buildx variable.
ARG TARGETARCH
# Copy the binary for the target architecture
COPY --from=builder /src/bin/my-service-${TARGETARCH} /bin/my-service

ENTRYPOINT ["/bin/my-service"]
