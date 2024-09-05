# syntax = docker/dockerfile:1.8
########################################

FROM registry.k8s.io/pause:3.8 AS pause

########################################
#
# Base image
#

FROM python AS base

ENV DEBIAN_FRONTEND=noninteractive TERM=xterm-color LC_ALL=C.UTF-8 LANG=C.UTF-8
ENV PYTHONUNBUFFERED=1 POETRY_VIRTUALENVS_CREATE=false

RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked \
    LC_ALL=C apt-get update -y && \
    LC_ALL=C apt-get install -y --no-install-recommends locales ca-certificates mime-support make libpq5 vim gettext procps && \
    sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen && LC_ALL=C locale-gen && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* && \
    useradd -ms /bin/bash --uid 5000 -d /app app

########################################
#
# Build the project with development dependencies
#

FROM base AS builder

RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-dev git && \
    pip3 install poetry && \
    rm -rf /var/lib/apt/lists/* /tmp/*

WORKDIR /www/app

COPY --chown=app:app ["myservice/poetry.lock","myservice/pyproject.toml","/www/app/"]
RUN --mount=type=cache,id=poetry,target=/root/.cache poetry install --no-interaction --no-root && \
    rm -rf /tmp/*

########################################
#
# Copy necessary files to the release image
#

FROM base AS release

ENV PYTHONPATH=/www/shared-apps PATH=/home/app/.local/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

COPY --from=pause /pause /pause
COPY --from=builder --chown=app:app /usr/local /usr/local
COPY                --chown=app:app myservice/ /www/app/

USER app
WORKDIR /www/app

CMD ["bash"]
