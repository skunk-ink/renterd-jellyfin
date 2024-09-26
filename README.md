# renterd-jellyfin

- [Introduction](#introduction)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
    - [Environment Variables](#environment-variables)
    - [Docker Compose Setup](#docker-compose-setup)
- [Usage](#usage)
    - [Clone the Repository](#clone-the-repository)
    - [Deploy the Docker Image](#deploy-the-docker-image)
    - [Accessing Jellyfin](#accessing-jellyfin)
    - [Updating the Container](#updating-the-container)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [License](#license)

# Introduction

This repository provides a custom [Jellyfin](https://jellyfin.org/) docker image with integrated [Rclone](https://rclone.org/) support to streamline setup with `renterd`. Providing a quick and easy method to begin streaming media stored on the Sia network. 

# Features

- **Jellyfin Integration:** Access and stream your `renterd` media library via Jellyfin's intuitive web interface and apps.
- **Rclone:** Mounts `renterd` through its S3 endpoint as `/mnt/renterd` within the container.
- **Plug n’ Play `renterd`:** Preconfigured to work out of the box with `renterd`. Streamlining the process of streaming media from the Sia network.
- **Persistent Storage:** Configuration and cache data are persisted using Docker volumes.
- **FUSE Support:** Enables filesystem mounting with appropriate permissions.

# Prerequisites

Before setting up the custom Jellyfin Docker container, ensure you have the following:

- **Docker:** Installed on your system. Install Docker
- **Docker Compose:** Installed to orchestrate multi-container setups. Install Docker Compose
- **`renterd` Instance:** A running [`renterd`](https://github.com/siafoundation/renterd) service with a configured S3 storage endpoint.

# Configuration

### Environment Variables

The custom Jellyfin Docker image requires several environment variables to configure Rclone and Jellyfin. These variables can be set in a `.env` file for ease of management.

**Configure `env.template` with your S3 details and rename to `.env` before deployment.**

```bash
# S3 Configuration
S3_BUCKET=your_bucket_name
S3_ACCESS_KEY_ID=your_access_key
S3_SECRET_ACCESS_KEY=your_secret_key
```

**Replace the placeholders** `your_bucket_name`, `your_access_key`, and `your_secret_key` with your `renterd` S3 details. You do not need to change the `S3_ENDPOINT_URL` unless you have changed `renterd`'s container name in your `docker-compose.yml` or would like to specify a different port.

### Docker Compose Setup

**Sample `docker-compose.yml`:**

```yaml
services:
  jellyfin:
    image: skunkink/renterd-jellyfin
    restart: always
    container_name: jellyfin
    environment:
      S3_BUCKET: ${S3_BUCKET:-default}
      S3_ENDPOINT_URL: ${S3_ENDPOINT_URL:-http://renterd:8080}
      S3_ACCESS_KEY_ID: ${S3_ACCESS_KEY_ID}
      S3_SECRET_ACCESS_KEY: ${S3_SECRET_ACCESS_KEY}
    ports:
      - 8096:8096
    volumes:
      - ./jellyfin/config:/config
      - ./jellyfin/cache:/cache
    cap_add:
      - SYS_ADMIN
    devices:
      - /dev/fuse
    security_opt:
      - apparmor:unconfined
```

If you are running `renterd` in docker, you can add the following to the end of your `jellyfin` service. This will ensure that `renterd` is running before attempting to create a S3 mount point.

```yaml
    depends_on:
      renterd:
        condition: service_started
```

# Usage

## Setup

### Clone the Repository

First, clone this repository to your local machine:

```bash
git clone https://github.com/skunk-ink/renterd-jellyfin.git
cd renterd-jellyfin
```

### Deploy the Docker Image

***Note:** Before you deploy the docker image, make sure you have created both a `.env` and `docker-compose.yml` as explained in the [configuration](#configuration) section.*

**Deploy the Jellyfin container using:**

```bash
docker compose up -d
```

**Explanation:**

- **`docker compose up -d`**: Builds the docker services as defined in the `docker-compose.yml` file.

  ***Ensure you're in the project directory containing the `docker-compose.yml` file.***

## Accessing Jellyfin

After successfully starting the services, access the Jellyfin web interface:

1. **Open a Web Browser:**
    
    Navigate to `http://localhost:8096` or `http://your_server_ip:8096`.
    
2. **Jellyfin Setup:**
    - **First-Time Setup:** Follow the on-screen instructions to set up your Jellyfin server.
    - **Media Libraries:** When adding libraries, your `renterd` media can be located under `/mnt/renterd`.

## Updating the Container

To update your running `renterd-jellyfin` container with the latest changes:

1. **Rebuild the Docker Image:**

```bash
docker compose pull
```

1. **Redeploy the Container:**

```bash
docker compose up -d
```

# Security Considerations

- **Environment Variables:**
    - Store sensitive information like your `S3_ACCESS_KEY_ID` and `S3_SECRET_ACCESS_KEY` securely in the `.env` file.
- **Network Exposure:**
    - Expose only necessary ports. If deploying in a production environment, consider using a reverse proxy with HTTPS.

# Troubleshooting

### Checking Logs

To inspect the logs of the Jellyfin container:

```bash
docker compose logs -f jellyfin
```

Press `Ctrl + C` to exit the log view.

# License

This project is licensed under the [MIT License](./LICENSE).