# syntax = docker/dockerfile:1.8
########################################

FROM --platform=${BUILDPLATFORM} golang:1.23-alpine3.20 AS builder
RUN apk update && apk add --no-cache make git
ENV GO111MODULE=on
WORKDIR /src

COPY ["go.mod", "/src"]
RUN go mod download && go mod verify

COPY . .
ARG TAG
ARG SHA
RUN make build-all-archs

########################################

FROM --platform=${TARGETARCH} scratch AS my-service
LABEL org.opencontainers.image.source="https://github.com/sergelogvinov/devops-examples" \
      org.opencontainers.image.licenses="MIT"

# To make one layer
COPY --from=gcr.io/distroless/static-debian12:nonroot . .

ARG TARGETARCH
COPY --from=builder /src/bin/my-service-${TARGETARCH} /bin/my-service

ENTRYPOINT ["/bin/my-service"]
