Purpose
=======

This repository is used to demonstrate an unexpected behaviour of Docker Compose.

Initial position
----------------

- A [Dockerfile](Dockerfile)
- A [Compose file](docker-compose.yml) with 2 services, namely:
    - A `local` service, which builds an image from the [Dockerfile](Dockerfile), without any `image:` tag
    - A `remote` service, which uses an image from a registry

Compose command
---------------

Run the following commands:

```bash
$ docker compose build
$ docker compose up -d --no-build --pull always
```

Expected behaviour
------------------

First command (`build`):

- The image for the `local` service is built in the first command (i.e. `build`)
- The image for the `remote` service is ignored, since it has not to be built

Second command (`up`):

- The image for the `remote` service is pulled (because of `--pull always`)
- The image for the `local` service isn't built (because of `--no-build`) or pulled (since there's no `image:` there)
- Both containers are started

Actual behaviour
----------------

No issues with the first command.

For the second command, Docker tries to pull the `local` image without having an `image:` tag defined at all:

```
$ docker compose up -d --pull always --no-build
[+] Running 2/2
 ✘ local Error 
 ✔ remote Pulled 
Error response from daemon: pull access denied for docker-pull-test-local, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

More testing
------------

Using only the `--no-build` flag works:

```
$ docker compose up -d --no-build
[+] Running 2/2
 ✔ Container docker-pull-test-local-1   Started
 ✔ Container docker-pull-test-remote-1  Started
```

Using only the `--pull always` flag works:

```
$ docker compose up -d --pull always
[+] Running 1/1
 ✔ remote Pulled
[+] Building 0.0s (0/0)
[+] Running 3/3
 ✔ Network docker-pull-test_default     Created
 ✔ Container docker-pull-test-local-1   Started
 ✔ Container docker-pull-test-remote-1  Started
```

Interestingly, `--no-build --pull policy` also fails:

```
$ docker compose up -d --pull policy --no-build
[+] Running 2/2
 ✘ local Error
 ✔ remote Pulled
Error response from daemon: pull access denied for docker-pull-test-local, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

Another bug
-----------

More interestingly, on another system, the `--pull policy` flag errors:

```
docker compose up -d --pull policy --no-build
invalid --pull option "policy"
```

The help states it should be working (it is on the other system):

```
 docker compose up -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker compose up [OPTIONS] [SERVICE...]

Create and start containers

Options:
~snip~
      --pull string               Pull image before running ("always"|"policy"|"never") (default "policy")
~snap~
```

Environment
-----------

I tried to run this on:

- Debian 12 (amd64)
- Docker version 24.0.7, build afdd53b
- Docker Compose version v2.21.0

The system where the `--pull policy` fails, is this:

- macOS 13.5.2 (arm)
- Docker version 24.0.6, build ed223bc
- Docker Compose version v2.23.0-desktop.1

GitHub issue
------------

I've created a [GitHub issue](https://github.com/docker/compose/issues/11236) for this problem.
