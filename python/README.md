# Build python project

Using make command:

```shell
make help
```

```shell
# Getting Started

To build this project locally, you must have the following installed:

- git
- make
- python
- docker
- docker-compose

help                           This help menu
clean                          Clean
lint                           Lint Code
unit                           Unit Tests
images                         Build images
```

### Build process

```shell
make build
```

### Testing process

```shell
make lint unit
```

### Release process

```shell
make images PUSH=true
```
