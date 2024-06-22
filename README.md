## Overview
This repo provides a Docker compose file for standing up a local lab
environment with the following resources:
- Artifactory repo

## Dependencies
Tested on Ubuntu 22.04:
```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo usermod -aG docker $USER
# Log out and log back in to be able to run docker as a non-root user
```
## Managing the Lab Environment
Stand up the lab:
```bash
docker compose up
```

Tear down the lab:
```bash
docker compose down
```

## Accessing Lab Services
The following services can be reached from the host machine by
targeting `localhost` (127.0.0.1) on the following TCP ports:
- Artifactory: 8082

The same services can be reached from other docker containers
by running them on the `lab_net` network and targeting the
service name.
