# Build python project

The Dockerfile was designed to build multi-architecture images efficiently.
Golang can build binaries for different architectures using the `GOOS` and `GOARCH` environment variables.
In first stage `builder` we build all binaries for different architectures and in the second stage `my-service` we copy the binary for the target architecture.

Read comments in the [Dockerfile](Dockerfile) for more details.

## Make commands

```shell
make help
```

```shell
# Getting Started

To build this project, you must have the following installed:

- git
- make
- golang 1.23+
- golangci-lint

help                           This help menu
clean                          Clean
build                          Build
lint                           Lint Code
unit                           Unit Tests
images                         Build images
```

### Build process

```shell
make build
```

<details>
<summary>Build output</summary>

```shell
CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 go build -ldflags "-w -s -X main.version=e3a75e5 -X main.commit=e3a75e5" \
                -o bin/my-service-arm64 ./cmd/my-service
```

</details>

### Testing process

```shell
make lint unit
```

<details>
<summary>Test output</summary>

```shell
```

</details>

### Release process

```shell
make images PUSH=true
```

<details>
<summary>Images output</summary>

```shell
docker buildx build --platform=linux/arm64 --output type=docker \
                --build-arg TAG=e3a75e5 \
                --build-arg SHA=e3a75e5 \
                -t ghcr.io/sergelogvinov/my-service:e3a75e5 \
                --target my-service \
                -f Dockerfile .
[+] Building 24.5s (18/18) FINISHED                                                                                                                                                                     docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                                                    0.0s
 => => transferring dockerfile: 801B                                                                                                                                                                                    0.0s
 => resolve image config for docker-image://docker.io/docker/dockerfile:1.8                                                                                                                                             1.1s
 => CACHED docker-image://docker.io/docker/dockerfile:1.8@sha256:e87caa74dcb7d46cd820352bfea12591f3dba3ddc4285e19c7dcd13359f7cefd                                                                                       0.0s
 => [internal] load metadata for gcr.io/distroless/static-debian12:nonroot                                                                                                                                              0.4s
 => [internal] load metadata for docker.io/library/golang:1.23-alpine3.20                                                                                                                                               1.4s
 => [internal] load .dockerignore                                                                                                                                                                                       0.0s
 => => transferring context: 153B                                                                                                                                                                                       0.0s
 => [builder 1/7] FROM docker.io/library/golang:1.23-alpine3.20@sha256:436e2d978524b15498b98faa367553ba6c3655671226f500c72ceb7afb2ef0b1                                                                                10.9s
 => => resolve docker.io/library/golang:1.23-alpine3.20@sha256:436e2d978524b15498b98faa367553ba6c3655671226f500c72ceb7afb2ef0b1                                                                                         0.0s
 => => sha256:a355a3cac949bed5cda9c62103ceb0f004727cedcd2a17d7c9836aea1a452fda 70.62MB / 70.62MB                                                                                                                        7.9s
 => => sha256:5c4c388ac50a370dc35feaa3a853b83dcb24a72d0a707d496c4a22a28459169b 126B / 126B                                                                                                                              0.3s
 => => sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1 32B / 32B                                                                                                                                0.4s
 => => sha256:436e2d978524b15498b98faa367553ba6c3655671226f500c72ceb7afb2ef0b1 10.29kB / 10.29kB                                                                                                                        0.0s
 => => sha256:7fc061b0c4e48920ba7fcd84e5eae946a6e9937727d6e192a09d8bf03542bdc8 1.92kB / 1.92kB                                                                                                                          0.0s
 => => sha256:9f13809c9794d3de14c9ea280a2300bbeedc8ca8fa20d48ff4a9260fac55a8de 2.09kB / 2.09kB                                                                                                                          0.0s
 => => extracting sha256:a355a3cac949bed5cda9c62103ceb0f004727cedcd2a17d7c9836aea1a452fda                                                                                                                               2.9s
 => => extracting sha256:5c4c388ac50a370dc35feaa3a853b83dcb24a72d0a707d496c4a22a28459169b                                                                                                                               0.0s
 => => extracting sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1                                                                                                                               0.0s
 => [internal] load build context                                                                                                                                                                                       0.0s
 => => transferring context: 246B                                                                                                                                                                                       0.0s
 => FROM gcr.io/distroless/static-debian12:nonroot@sha256:42d15c647a762d3ce3a67eab394220f5268915d6ddba9006871e16e4698c3a24                                                                                              0.0s
 => [builder 2/7] RUN apk update && apk add --no-cache make git                                                                                                                                                         2.6s
 => [builder 3/7] WORKDIR /src                                                                                                                                                                                          0.0s
 => [builder 4/7] COPY [go.mod, /src]                                                                                                                                                                                   0.0s
 => [builder 5/7] RUN go mod download && go mod verify                                                                                                                                                                  0.1s
 => [builder 6/7] COPY . .                                                                                                                                                                                              0.0s
 => [builder 7/7] RUN make build-all-archs                                                                                                                                                                              8.0s
 => CACHED [my-service 1/2] COPY --from=gcr.io/distroless/static-debian12:nonroot . .                                                                                                                                   0.0s
 => [my-service 2/2] COPY --from=builder /src/bin/my-service-arm64 /bin/my-service                                                                                                                                      0.0s
 => exporting to image                                                                                                                                                                                                  0.0s
 => => exporting layers                                                                                                                                                                                                 0.0s
 => => writing image sha256:0e6cc15597670fe85e6be0b25ce8472430dc7433bf8f918e8ab4ce78294b929b                                                                                                                            0.0s
 => => naming to ghcr.io/sergelogvinov/my-service:e3a75e5
```

</details>
