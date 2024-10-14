# Devops opinionated best practices

I've been working in the DevOps field for a while now and notice that many companies are repeating the same mistakes. This document is a collection of best practices that I've found to be useful in my work. It's opinionated, and it's not meant to be a comprehensive guide to DevOps. Instead, it's a collection of practices that I've found to be useful in my work.

All ideas are working for mono-repo projects and for microservices.

## Table of Contents

- [Build projects](#build-projects)
- [Continuous Integration](#continuous-integration)
- [Tools](#tools)

Project examples:

- [Golang](golang/)
- [Python](python/)
- [Symfony](symfony/) PHP-fpm, Nginx

## Build projects

Since `docker` is the most popular containerization tool, I recommend using it for building projects. It's easy to use, and it's widely supported. Docker has alternatives like Podman, but most of the code are compatible with Docker. I prefer the `BuildKit` extension for Docker because it's faster and more efficient than the standard Docker build process. Also, it's supported multi architecture builds.

Buildkit allows you to use cache volumes, which is useful for caching dependencies between builds.
It's faster than the standard Docker build process because it doesn't need to download the dependencies every time you build the project.
Think of the images layers as a cache. If you change the code, the layer with the code will be invalidated, and the next layer will be rebuilt.

Here's a simple Dockerfile that you can use to build your projects:

```Dockerfile
# syntax = docker/dockerfile:1.8
########################################

FROM registry.k8s.io/pause:3.8 AS pause

########################################
#
# Base image
#

FROM python AS base

# Basic requirements for the environment
ENV DEBIAN_FRONTEND=noninteractive TERM=xterm-color LC_ALL=C.UTF-8 LANG=C.UTF-8
ENV PYTHONUNBUFFERED=1 POETRY_VIRTUALENVS_CREATE=false

# Install basic packages and create a non-root user
RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked \
    LC_ALL=C apt-get update -y && \
    LC_ALL=C apt-get install -y --no-install-recommends locales ca-certificates mime-support make libpq5 vim gettext procps && \
    sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen && LC_ALL=C locale-gen && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* && \
    useradd -ms /bin/bash --uid 5000 -d /www/app app

########################################
#
# Build the project with development dependencies
#

FROM base AS builder

# Dependencies for building the python packages
RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-dev git && \
    pip3 install poetry && \
    rm -rf /var/lib/apt/lists/* /tmp/*

WORKDIR /www/app

# Install the project dependencies
# They don't change often, so we can cache them
COPY --chown=app:app ["myservice/poetry.lock","myservice/pyproject.toml","/www/app/"]
RUN --mount=type=cache,id=poetry,target=/root/.cache poetry install --no-interaction --no-root && \
    rm -rf /tmp/*

########################################
#
# Copy necessary files to the release image
#

FROM base AS release

ENV PYTHONPATH=/www/shared-apps PATH=/home/app/.local/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Pause binary for run it for testing purposes
# It shuts down immediately, and has small size
COPY --from=pause /pause /pause
# Python packages
COPY --from=builder --chown=app:app /usr/local /usr/local
# Project files
COPY                --chown=app:app myservice/ /www/app/

# Switch to the non-root user
USER app
WORKDIR /www/app

CMD ["bash"]
```

## Continuous Integration

There are many CI/CD tools available, but all of them have they own vendor lock-in. Which means that if you start using a specific tool, it's hard to switch to another CICD. But they all have the same basic features, like bash, docker, and git. So, better to use shell scripts for your CI/CD pipelines and store them in your repository. Just run the shell scripts in your CI/CD specific rulesets.

### Build process

How to shoose the right name of the shell scripts?
1. The world get used to the `Makefile`, `Taskfile`. Makefile is the most popular, but it's not easy to read and write. So, I recommend using the `Taskfile` format. It's easy to read and write, and it's supported by most of the CI/CD tools.
1. Put the scripts in the root of the repository. In mono-repo projects, store the scripts in project folders, and one script on the root of the repository to build all the projects.
1. Create a `README.md` file in the root of the repository and explain how to run the scripts (basic commands).
1. Do not forget to add `help` command to the scripts. It's useful for the developers to understand what the script does.

Example:

`make help` output:

```shell
# Getting Started

To build this project, you must have the following installed:

- git
- make
- golang 1.20+
- golangci-lint

help                           This help menu
clean                          Clean
build                          Build
run                            Run
lint                           Lint Code
unit                           Unit Tests
```

### Testing process

If you code depends on other services, like databases, queues, etc., you should use the `docker-compose` tool to run the services in the CI/CD pipeline. It's easy to use, and do not forget to add `health checks` to the services. It's useful to wait for the services to be ready before running the tests.

To simplify the process, I recommend creating a `base` service in the `docker-compose` file. It's a service that creates a network for the other services. So, all services will use localhost to connect to each other.

Example:

```yaml
# docker-compose.yml

services:
  # It creates a network for the services
  base:
    image: registry.k8s.io/pause:3.8

  postgres:
    image: ghcr.io/sergelogvinov/postgresql:15.6
    shm_size: 1g
    # It uses the network created by the base service
    network_mode: "service:base"
    # Disable fsync to speed up the tests
    command: -c fsync=off
    # Default user and password
    environment:
      - POSTGRES_USER=myservice
      - POSTGRES_PASSWORD=myservice
      - POSTGRES_DB=myservice

    # Docker will wait for the service to be ready before continuing (depends_on)
    healthcheck:
      test: ["CMD-SHELL", "psql -U myservice -d myservice -c 'SELECT 1'"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 10s

  test:
    build:
      context: .
      target: test
      dockerfile: myservice/Dockerfile
    # It uses the network created by the base service
    network_mode: "service:base"
    # Run the container, than we can run the tests inside the container
    command: /pause
    depends_on:
      - postgres
```

Build and run the tests, will look like this:

```shell
docker compose -f docker-compose.yml build
docker compose -f docker-compose.yml up -d --wait
docker compose -f docker-compose.yml exec test my-project-test
```

## Tools

* [Docker](https://www.docker.com/)
* [BuildKit](https://github.com/moby/buildkit)
* [Docker Compose](https://docs.docker.com/compose/)
* [Dive](https://github.com/wagoodman/dive)
* [Taskfile](https://taskfile.dev/)
* [Makefile](https://www.gnu.org/software/make/)

Self hosted CI/CD agents/conrollers:

* [TeamCity](https://github.com/sergelogvinov/containers/tree/main/teamcity)
* [Githab Actions](https://github.com/sergelogvinov/containers/tree/main/github-actions-runner)

## Other resources

* [Understanding Docker](https://dev.to/aurelievache/understanding-docker-part-1-retrieve-pull-images-3ccn)
