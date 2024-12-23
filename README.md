## Overview

This project provides a Docker Compose file to host a private DevOps environment
from a single server for situations such as a typical home network.

The individual DevOps resources hosted on the server can then be accessed over
HTTPS (and SSH for Gitlab) simply through URL path prefixes such as:

- `https://<SERVER_HOSTNAME>/gitlab`
- `https://<SERVER_HOSTNAME>/jenkins`
- `https://<SERVER_HOSTNAME>/nexus`

This setup was tested on Ubuntu 24.04.

## Dependencies

- Docker
- Docker Compose

## Assumptions/Limitations

The following assumptions/limitations are made:
- The network is not publicly accessible.
- All DevOps resources are hosted on a single server.
- All DevOps resources will use an automatically generated self-signed cert to
  allow secure communications via HTTPS (although the cert will not be validated
  by a CA since the network is assumed not to be publicly accessible).

## Hosted Resources

The following resources are created:
- Gitlab repository for source code.
- Jenkins automation server for CI/CD.
- Nexus repository for managing artifacts, binaries, and containers.

## Setup

1. Install the following packages on your server:
    - Docker
    - Docker Compose
2. Follow the [Docker post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
   to allow running Docker as a non-root user.
3. Allow incoming traffic for the following ports on the server's firewall
   (if allowing external access from your local network):
    - TCP port 22 (SSH)
    - TCP port 443 (HTTPS)
4. The gitlab container exposes the SSH port for secure communications.
   If the host server is also running an SSH server, either:
    - Modify the `compose.yaml` to stop/expose a different port for the SSH
      service.
    - Stop/disable the host's SSH server
    - Modify the host's SSH server configs to listen on a different port

## Run

The DevOps resources can be started using the following command template
(substitute `<HOSTNAME>` with your server's hostname):

```bash
HOSTNAME=<HOSTNAME> \
DOCKER_GID=$(getent group docker | cut -d: -f3) \
docker compose up --detach
```

Example command for a server with the hostname `devops.home.arpa`:
```bash
HOSTNAME=devops.home.arpa \
DOCKER_GID=$(getent group docker | cut -d: -f3) \
docker compose up --detach
```

### Environment Variables

As shown in the example above, running the Docker compose file requires certain
environment variables to be defined:
- `HOSTNAME` (required): The hostname of the server to be used as the base
                         URL for all resources.
- `DOCKER_GID` (optional): The ID of the `docker` group on the host machine.
                           This allows the Jenkins master node to spawn sibling
                           Docker containers on the host machine if executing
                           pipeline jobs that run Docker containers.

### Initial passwords

The initial passwords for each resource can be found in the standard locations
inside their respective containers:
- `docker exec gitlab cat /etc/gitlab/initial_root_password`
- `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`
- `docker exec nexus cat /nexus-data/admin.password`

## Stop

The containers can be stopped/destroyed with the following command template
(substitute `<HOSTNAME>` with your server's hostname):

```bash
HOSTNAME=<HOSTNAME> docker compose down
```

## Volumes

The data in the containers are stored in Docker volumes and persist even when
the containers are destroyed. To completely reset a resource, destroy the
respective volume(s) by:

- Stop and remove running containers: `HOSTNAME=<HOSTNAME> docker compose down`
- Destroy the desired volume(s):
    - Gitlab:
        - `docker volume rm devops_gitlab_config`
        - `docker volume rm devops_gitlab_logs`
        - `docker volume rm devops_gitlab_data`
    - Jenkins: `docker volume rm devops_jenkins`
    - Nexus: `docker volume rm devops_nexus`

## Updates

The Docker Compose file always pulls the latest Docker images every time
`docker compose up` is run so to update the DevOps resources to the latest
versions, simply:
1. Stop and destroy the running containers using the instructions in the
   **Stop** section.
2. Recreate the containers using the instructions in the **Run** section.
