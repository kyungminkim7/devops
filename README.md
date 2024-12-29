# DevOps

This project provides a Docker Compose file to host a private DevOps environment
from a single server for situations such as a typical home network.

The individual DevOps resources hosted on the server can then be accessed over
HTTPS (and SSH for Gitlab) simply through URL path prefixes such as:

- `https://<SERVER_HOSTNAME>/gitlab`
- `https://<SERVER_HOSTNAME>/nexus`

This setup was tested on Ubuntu 24.04.

## Dependencies

- Docker
- Docker Compose

## Assumptions/Limitations

The following assumptions/limitations are made:

- The network is not publicly accessible.
- All DevOps resources are hosted on a single server.
- A certificate will be provided for HTTPS/TLS communications

## Hosted Resources

The following resources are created:

- Gitlab repository for source code management and CI/CD pipeline.
- Gitlab runner for CI/CD.
- Nexus repository for managing artifacts, binaries, and containers.

## Setup

1. Install the following packages on your server:
    - Docker
    - Docker Compose
2. Follow the [Docker post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
   to allow running Docker as a non-root user.
3. The gitlab container exposes the SSH port for secure communications.
   If the host server is also running an SSH server, either:
    - Modify the `compose.yaml` to stop/expose a different port for the SSH
      service.
    - Stop/disable the host's SSH server
    - Modify the host's SSH server configs to listen on a different port
4. Create an SSL key and certificate named `ca.key` and `ca.crt` in the
   `certs` subdirectory of this project:

   ```bash
   openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -noenc \
       -keyout certs/ca.key -out certs/ca.crt \
       -addext 'subjectAltName = DNS:<HOSTNAME>'
   ```

## Run

The DevOps resources can be started using the following command template
(substitute `<HOSTNAME>` with your server's hostname and domain name if you
have one):

```bash
HOSTNAME=<HOSTNAME> docker compose up --detach
```

Example for a server with the hostname `devops` on the `home.arpa` domain:

```bash
HOSTNAME=devops.home.arpa docker compose up --detach
```

### Initial passwords

The initial passwords for each resource can be found in the standard locations
inside their respective containers:

- `docker exec gitlab cat /etc/gitlab/initial_root_password`
- `docker exec nexus cat /nexus-data/admin.password`

### Registering Gitlab runner

A Gitlab runner container capable of running a Docker executor is automatically
created. This runner can be accessed to register against the Gitlab server
container using the following command:

```bash
docker exec -it runner bash
```

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
  - Gitlab runner: `docker volume rm devops_gitlab_runner`
  - Nexus: `docker volume rm devops_nexus`

## Updates

The Docker Compose file always pulls the latest Docker images every time
`docker compose up` is run so to update the DevOps resources to the latest
versions, simply:

1. Stop and destroy the running containers using the instructions in the
   **Stop** section.
2. Recreate the containers using the instructions in the **Run** section.
