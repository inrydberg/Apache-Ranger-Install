# Apache Ranger Docker Deployment

This repository contains configuration for deploying Apache Ranger 2.4.0 using Docker **specifically for x86_64/AMD64 systems.**

Due to the current availability of Apache Ranger 2.4.0 Docker images being only for ARM64, this guide includes necessary steps for running these images on x86_64 architecture using emulation.
If you're running on ARM64 architecture (like Apple Silicon), you should follow the simpler deployment process described in the official Apache Ranger Docker Hub repository: https://hub.docker.com/r/apache/ranger

## Prerequisites

- Docker Engine
- Docker Compose

## Setup Instructions

1. Clone this repository or copy the .yml file contents into a new docker-compose.yml file

2. Set up Docker BuildX for multi-architecture support:
```bash
# Create a new builder instance
docker buildx create --name multiarch-builder --driver docker-container --use

# Verify the builder is running
docker buildx inspect --bootstrap
```

3. Enable QEMU for platform emulation:
```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```

4. Start the services:
```bash
DOCKER_DEFAULT_PLATFORM=linux/arm64/v8 docker-compose up -d
```

5. Monitor the initialization process:
```bash
docker logs -f ranger
```

Wait until you see the message:
```
Apache Ranger Admin Service with pid XXXX has started.
```

6. Reset the admin password (required for first login):
```bash
# Connect to the database container
docker exec -it ranger-db bash

# Connect to PostgreSQL
psql -U rangeradmin -d ranger

# Reset admin password with the following SQL command
UPDATE x_portal_user 
SET password = 'ceb4f32325eda6142bd65215f4c0f371', 
    password_updated_time = now() 
WHERE login_id = 'admin';

# Exit PostgreSQL
\q

# Exit container
exit
```

7. Access the Ranger Admin UI:
- URL: http://localhost:6080 (or your server IP)
- Username: `admin`
- Password: `admin`

## Container Management

To stop the services:
```bash
docker-compose down
```

To completely clean up (including volumes):
```bash
docker-compose down -v
```

To view logs of specific services:
```bash
docker logs ranger        # Ranger Admin logs
docker logs ranger-db     # Database logs
docker logs ranger-solr   # Solr logs
docker logs ranger-zk     # ZooKeeper logs
```

## Troubleshooting

1. If you see platform mismatch errors:
   - Verify that QEMU is properly installed
   - Make sure you're using the DOCKER_DEFAULT_PLATFORM environment variable when starting services

2. If the web UI is inaccessible:
   - Check if all containers are running: `docker ps -a`
   - Verify logs for any errors: `docker logs ranger`
   - Ensure port 6080 is not being used by another service

3. If you can't log in:
   - Verify you've completed the password reset step
   - Check if the ranger-db container is healthy

## Architecture

The deployment consists of four main components:
- Apache Ranger Admin (UI and core services)
- PostgreSQL Database (metadata storage)
- Apache Solr (audit storage)
- ZooKeeper (coordination service)

## Notes

- This deployment uses ARM64 images with x86_64 emulation due to Apache Ranger 2.4.0 Docker images being available only for ARM64
- The setup includes proper health checks and container dependencies
- Default ports used:
  - 6080: Ranger Admin UI
  - 8983: Solr Admin
  - 2181: ZooKeeper

## Contributing

Feel free to submit issues and pull requests for improvements to the configuration.
