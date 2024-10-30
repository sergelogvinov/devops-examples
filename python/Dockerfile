# syntax = docker/dockerfile:1.10
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
ARG TARGETARCH
RUN --mount=type=cache,id=apt-cache-python-$TARGETARCH,target=/var/cache/apt,sharing=locked \
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

# Create a cache for specific build architecture
ARG TARGETARCH
# Make possible to create new cache, if the pip packages have errors or need to be updated
ARG EPOCH="0"
# Dependencies for building the python packages
RUN --mount=type=cache,id=apt-cache-python-$TARGETARCH,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-dev git
RUN --mount=type=cache,id=pip-$TARGETARCH-$EPOCH,target=/root/.cache/pip \
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
