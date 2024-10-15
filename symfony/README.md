# Build symfony project

The Dockerfile was designed as a multi-stage build to build the symfony project efficiently.

* the `base` stage has all the dependencies to run nginx and php-fpm.
* the `builder` stage is used to build the symfony project and all modules.
* the `release` stage is final image with the symfony project.

We split the build process into multiple stages to reduce the final image size.
Read comments in the [Dockerfile](Dockerfile) for more details.

## Make commands

```shell
make help
```

```shell
# Getting Started

To build this project locally, you must have the following installed:

- git
- make
- docker
- docker-compose

help                           This help menu
clean                          Clean
lint                           Lint Code
unit                           Unit Tests
images                         Build images
```
