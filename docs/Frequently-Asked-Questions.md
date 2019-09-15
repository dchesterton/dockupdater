# Frequently Asked Questions

- [Frequently Asked Questions](#frequently-asked-questions)
  - [How to configure Timezone](#how-to-configure-timezone)
  - [How to stop Dockupdater auto update](#how-to-stop-dockupdater-auto-update)
  - [How install Dockupdater without docker](#how-install-dockupdater-without-docker)
  - [How to remove containers after service update](#how-to-remove-containers-after-service-update)
  - [How to use with service but without stack](#how-to-use-with-service-but-without-stack)
  - [How to use special token {stack}](#how-to-use-special-token-stack)

***

## How to configure Timezone

To more closely monitor dockupdater actions and for accurate log ingestion, you can change the timezone of the container from UTC by setting the [`TZ`](http://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) environment variable like so:

```bash
docker run -d --name dockupdater \
  -e TZ=America/Chicago \
  -v /var/run/docker.sock:/var/run/docker.sock \
  dockupdater/dockupdater
```

## How to stop Dockupdater auto update

Your can add label `dockupdater.disable="true"` on the container or service to disable auto update.

If your run a standalone container for dockupdater:

```bash
docker run -d --name dockupdater \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --label dockupdater.disable="true"
  dockupdater/dockupdater
```

If your run dockupdater with a stack:

```bash
version: "3.6"

services:
  dockupdater:
    image: dockupdater/dockupdater
    deploy:
      labels:
        dockupdater.disable: "true"
```

## How install Dockupdater without docker

Dockupdater can also be installed via pip:

```bash
pip install dockupdater
```

And can then be invoked using the dockupdater command:

$ dockupdater --interval 300 --log-level debug

> Dockupdater need Python 3.6 or up

## How to remove containers after service update

By default Docker swarm keep 5 stop containers by service. You can configure that number at 0 to always remove old containers.

The update your docker swarm, run that command on a manager:

$ docker swarm update --task-history-limit=0

## How to use with service but without stack

You can start a service with the command line without using stack file :

```bash
docker service create --name test1 --replicas=2 --tty=true busybox
```

Unfortunately that cause an issue with Dockupdater. They have 2 workarounds:

**Option 1**

Run Dockupdater with the option [`--disable-containers-check`](Options.md#disable-containers-check). That will disable update for standalone containers.

**Option 2**

Add label [`dockupdater.disable`](Labels.md#disable-update) on the containers, this will disable update for standalone containers, but service will be updated normally.

You can add label for all containers of a service like that:
```bash
docker service create --name test1 --replicas=2 --tty=true --container-label="dockupdater.disable=true" busybox
```

## How to use special token {stack}

When you run use services, you can use a token {stack} with the [`--start`](Options.md#start) and [`--stop`](Options.md#stop) options.

`dockupdater.yml`

```bash
version: "3.6"

services:
  dockupdater:
    image: dockupdater/dockupdater
    deploy:
      placement:
        constraints:
          - node.role == manager
```

`stack.yml`

```bash
  service1:
    image: image1:${TAG}
    deploy:
      labels:
        STOPS: "{stack}_service2"
        STARTS: "{stack}_service2"

  service2:
    image: image2:${TAG}
```

And start all stack:

```bash
docker stack deploy -c dockupdater.yml dockupdater
TAG=latest docker stack deploy -c stack.yml test1
TAG=test1 docker stack deploy -c stack.yml test2
TAG=test2 docker stack deploy -c stack.yml test3
```

On update of `image1` this will restart `service2`, but only on the same stack of the `image1` update.
