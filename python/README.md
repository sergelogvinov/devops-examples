# Build python project

The Dockerfile has multi-stage build to build the image efficiently.
- `base` stage installs the required basic dependencies and configures the locale.
- `builder` stage installs the required dependencies for building the project.
- `release` stage copies the required files and python packages to the final image.

Read comments in the [Dockerfile](Dockerfile) for more details.

## Make commands

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

<details>
<summary>Build output</summary>

```shell
[+] Building 1.4s (20/20) FINISHED                                                                                                                                                                      docker:desktop-linux
 => [unittest internal] load build definition from Dockerfile                                                                                                                                                           0.0s
 => => transferring dockerfile: 1.86kB                                                                                                                                                                                  0.0s
 => [unittest] resolve image config for docker-image://docker.io/docker/dockerfile:1.8                                                                                                                                  1.2s
 => CACHED [unittest] docker-image://docker.io/docker/dockerfile:1.8@sha256:e87caa74dcb7d46cd820352bfea12591f3dba3ddc4285e19c7dcd13359f7cefd                                                                            0.0s
 => [unittest internal] load metadata for registry.k8s.io/pause:3.8                                                                                                                                                     0.0s
 => [unittest internal] load metadata for docker.io/library/python:latest                                                                                                                                               0.0s
 => [unittest internal] load .dockerignore                                                                                                                                                                              0.0s
 => => transferring context: 153B                                                                                                                                                                                       0.0s
 => [unittest pause 1/1] FROM registry.k8s.io/pause:3.8                                                                                                                                                                 0.0s
 => [unittest base 1/2] FROM docker.io/library/python:latest                                                                                                                                                            0.0s
 => [unittest internal] load build context                                                                                                                                                                              0.0s
 => => transferring context: 2.04kB                                                                                                                                                                                     0.0s
 => CACHED [unittest base 2/2] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     LC_ALL=C apt-get update -y &&     LC_ALL=C apt-get install -y --no-install-recommends locales ca-ce  0.0s
 => CACHED [unittest release 1/4] COPY --from=pause /pause /pause                                                                                                                                                       0.0s
 => CACHED [unittest builder 1/4] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-d  0.0s
 => CACHED [unittest builder 2/4] WORKDIR /www/app                                                                                                                                                                      0.0s
 => CACHED [unittest builder 3/4] COPY --chown=app:app [myservice/poetry.lock,myservice/pyproject.toml,/www/app/]                                                                                                       0.0s
 => CACHED [unittest builder 4/4] RUN --mount=type=cache,id=poetry,target=/root/.cache poetry install --no-interaction --no-root &&     rm -rf /tmp/*                                                                   0.0s
 => CACHED [unittest release 2/4] COPY --from=builder --chown=app:app /usr/local /usr/local                                                                                                                             0.0s
 => CACHED [unittest release 3/4] COPY                --chown=app:app myservice/ /www/app/                                                                                                                              0.0s
 => CACHED [unittest release 4/4] WORKDIR /www/app                                                                                                                                                                      0.0s
 => [unittest] exporting to image                                                                                                                                                                                       0.0s
 => => exporting layers                                                                                                                                                                                                 0.0s
 => => writing image sha256:cd7064235998d7493a1de3bfae0cbb4f1ae74ea0b03a851d1c748bc0c8c627d9                                                                                                                            0.0s
 => => naming to docker.io/library/python-unittest                                                                                                                                                                      0.0s
 => [unittest] resolving provenance for metadata file
```

</details>

### Testing process

```shell
make lint unit
```

<details>
<summary>Test output</summary>

```shell
[+] Building 0.7s (20/20) FINISHED                                                                                                                                                                      docker:desktop-linux
 => [unittest internal] load build definition from Dockerfile                                                                                                                                                           0.0s
 => => transferring dockerfile: 1.86kB                                                                                                                                                                                  0.0s
 => [unittest] resolve image config for docker-image://docker.io/docker/dockerfile:1.8                                                                                                                                  0.5s
 => CACHED [unittest] docker-image://docker.io/docker/dockerfile:1.8@sha256:e87caa74dcb7d46cd820352bfea12591f3dba3ddc4285e19c7dcd13359f7cefd                                                                            0.0s
 => [unittest internal] load metadata for docker.io/library/python:latest                                                                                                                                               0.0s
 => [unittest internal] load metadata for registry.k8s.io/pause:3.8                                                                                                                                                     0.0s
 => [unittest internal] load .dockerignore                                                                                                                                                                              0.0s
 => => transferring context: 153B                                                                                                                                                                                       0.0s
 => [unittest internal] load build context                                                                                                                                                                              0.0s
 => => transferring context: 2.04kB                                                                                                                                                                                     0.0s
 => [unittest pause 1/1] FROM registry.k8s.io/pause:3.8                                                                                                                                                                 0.0s
 => [unittest base 1/2] FROM docker.io/library/python:latest                                                                                                                                                            0.0s
 => CACHED [unittest base 2/2] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     LC_ALL=C apt-get update -y &&     LC_ALL=C apt-get install -y --no-install-recommends locales ca-ce  0.0s
 => CACHED [unittest release 1/4] COPY --from=pause /pause /pause                                                                                                                                                       0.0s
 => CACHED [unittest builder 1/4] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-d  0.0s
 => CACHED [unittest builder 2/4] WORKDIR /www/app                                                                                                                                                                      0.0s
 => CACHED [unittest builder 3/4] COPY --chown=app:app [myservice/poetry.lock,myservice/pyproject.toml,/www/app/]                                                                                                       0.0s
 => CACHED [unittest builder 4/4] RUN --mount=type=cache,id=poetry,target=/root/.cache poetry install --no-interaction --no-root &&     rm -rf /tmp/*                                                                   0.0s
 => CACHED [unittest release 2/4] COPY --from=builder --chown=app:app /usr/local /usr/local                                                                                                                             0.0s
 => CACHED [unittest release 3/4] COPY                --chown=app:app myservice/ /www/app/                                                                                                                              0.0s
 => CACHED [unittest release 4/4] WORKDIR /www/app                                                                                                                                                                      0.0s
 => [unittest] exporting to image                                                                                                                                                                                       0.0s
 => => exporting layers                                                                                                                                                                                                 0.0s
 => => writing image sha256:cd7064235998d7493a1de3bfae0cbb4f1ae74ea0b03a851d1c748bc0c8c627d9                                                                                                                            0.0s
 => => naming to docker.io/library/python-unittest                                                                                                                                                                      0.0s
 => [unittest] resolving provenance for metadata file                                                                                                                                                                   0.0s
[+] Running 4/4
 ✔ Network python_default       Created                                                                                                                                                                                 0.0s
 ✔ Container python-base-1      Healthy                                                                                                                                                                                 0.7s
 ✔ Container python-postgres-1  Healthy                                                                                                                                                                                 5.7s
 ✔ Container python-unittest-1  Healthy                                                                                                                                                                                 0.7s
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
/Library/Developer/CommandLineTools/usr/bin/make -f Makefile unit-stop
[+] Running 4/4
 ✔ Container python-unittest-1  Removed                                                                                                                                                                                 0.1s
 ✔ Container python-postgres-1  Removed                                                                                                                                                                                 0.0s
 ✔ Container python-base-1      Removed                                                                                                                                                                                 0.1s
 ✔ Network python_default       Removed
```

</details>

### Release process

```shell
make images PUSH=true
```

<details>
<summary>Images output</summary>

```shell
 => [internal] load build definition from Dockerfile                                                                                                                                                                    0.0s
 => => transferring dockerfile: 1.86kB                                                                                                                                                                                  0.0s
 => resolve image config for docker-image://docker.io/docker/dockerfile:1.8                                                                                                                                             0.5s
 => CACHED docker-image://docker.io/docker/dockerfile:1.8@sha256:e87caa74dcb7d46cd820352bfea12591f3dba3ddc4285e19c7dcd13359f7cefd                                                                                       0.0s
 => [internal] load metadata for docker.io/library/python:latest                                                                                                                                                        0.0s
 => [internal] load metadata for registry.k8s.io/pause:3.8                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                                       0.0s
 => => transferring context: 153B                                                                                                                                                                                       0.0s
 => [internal] load build context                                                                                                                                                                                       0.0s
 => => transferring context: 2.04kB                                                                                                                                                                                     0.0s
 => [base 1/2] FROM docker.io/library/python:latest                                                                                                                                                                     0.0s
 => [pause 1/1] FROM registry.k8s.io/pause:3.8                                                                                                                                                                          0.0s
 => CACHED [base 2/2] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     LC_ALL=C apt-get update -y &&     LC_ALL=C apt-get install -y --no-install-recommends locales ca-certificate  0.0s
 => CACHED [release 1/4] COPY --from=pause /pause /pause                                                                                                                                                                0.0s
 => CACHED [builder 1/4] RUN --mount=type=cache,id=apt-cache-python,target=/var/cache/apt,sharing=locked     apt-get update && apt-get install -y --no-install-recommends build-essential python3-dev libpq-dev git &&  0.0s
 => CACHED [builder 2/4] WORKDIR /www/app                                                                                                                                                                               0.0s
 => CACHED [builder 3/4] COPY --chown=app:app [myservice/poetry.lock,myservice/pyproject.toml,/www/app/]                                                                                                                0.0s
 => CACHED [builder 4/4] RUN --mount=type=cache,id=poetry,target=/root/.cache poetry install --no-interaction --no-root &&     rm -rf /tmp/*                                                                            0.0s
 => CACHED [release 2/4] COPY --from=builder --chown=app:app /usr/local /usr/local                                                                                                                                      0.0s
 => CACHED [release 3/4] COPY                --chown=app:app myservice/ /www/app/                                                                                                                                       0.0s
 => CACHED [release 4/4] WORKDIR /www/app                                                                                                                                                                               0.0s
 => exporting to image                                                                                                                                                                                                  0.0s
 => => exporting layers                                                                                                                                                                                                 0.0s
 => => writing image sha256:641e01066665943482abcf72f102e30bf6ae559f939b1ae0f1edd5555b2db753                                                                                                                            0.0s
 => => naming to ghcr.io/sergelogvinov/release:8cd3b14
```

</details>
