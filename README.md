# renterd-jellyfin

- [Introduction](#introduction)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
    - [1. Clone the Repository](#1-clone-the-repository)
    - [2. Deploy the Docker Image](#2-deploy-the-docker-image)
- [Configuration](#configuration)
    - [Environment Variables](#environment-variables)
    - [Docker Compose Setup with `renterd`](#docker-compose-setup-with-renterd)
- [Usage](#usage)
    - [Starting the Services](#starting-the-services)
    - [Accessing Jellyfin](#accessing-jellyfin)
- [Volumes and Data Persistence](#volumes-and-data-persistence)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Updating the Container](#updating-the-container)
- [License](#license)

## Introduction

This repository provides a custom [Jellyfin](https://jellyfin.org/) docker image with integrated [Rclone](https://rclone.org/) support to streamline setup with `renterd`. Providing a quick and easy method to begin streaming media stored on the Sia network. 

## Features

- **Jellyfin Integration:** Access and stream your `renterd` media library via Jellyfin's intuitive web interface and apps.
- **Rclone:** Mounts `renterd` through its S3 endpoint as `/mnt/renterd` within the container.
- **Plug n’ Play `renterd`:** Preconfigured to work out of the box with `renterd`. Streamlining the process of streaming media from the Sia network.
- **Persistent Storage:** Configuration and cache data are persisted using Docker volumes.
- **FUSE Support:** Enables filesystem mounting with appropriate permissions.

## Prerequisites

Before setting up the custom Jellyfin Docker container, ensure you have the following:

- **Docker:** Installed on your system. Install Docker
- **Docker Compose:** Installed to orchestrate multi-container setups. Install Docker Compose
- **`renterd` Instance:** A running [`renterd`](https://github.com/siafoundation/renterd) service with a configured S3 storage endpoint.

## Installation

### 1. Clone the Repository

First, clone this repository to your local machine:

```bash
git clone https://github.com/skunk-ink/renterd-jellyfin.git
cd renterd-jellyfin
```

### 2. Deploy the Docker Image

Deploy the Jellyfin container:

```bash
docker compose up -d
```

**Explanation:**

- **`docker compose up -d`**: Builds the docker services as defined in the `docker-compose.yml` file.

*Ensure you're in the project directory containing the `docker-compose.yml` file.*

## Configuration

### Environment Variables

The custom Jellyfin Docker image requires several environment variables to configure Rclone and Jellyfin. These variables can be set in a `.env` file for ease of management.

**Configure `env.template` with your S3 details and rename to `.env` before deployment.**

```bash
# S3 Configuration
S3_BUCKET=your_bucket_name
S3_ACCESS_KEY_ID=your_access_key
S3_SECRET_ACCESS_KEY=your_secret_key
```

**Replace the placeholders** `your_bucket_name`, `your_access_key`, and `your_secret_key` with the S3 details you configured in `renterd`. You do not need to change the `S3_ENDPOINT_URL` unless you have changed `renterd`'s container name in your `docker-compose.yml`.

### Docker Compose Setup with `renterd`

The provided `docker-compose.yml` orchestrates the `renterd` and `jellyfin` services:

**Sample `docker-compose.yml`:**

```yaml
services:
  renterd:
    image: ghcr.io/siafoundation/renterd
    restart: always
    container_name: renterd
    volumes:
      - ./renterd/:/data
    ports:
      - 9980:9980
      - 8080:8080

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
    depends_on:
      renterd:
        condition: service_started
```

## Usage

### Starting the Services

Once you've configured the `.env` and `docker-compose.yml` files, start the services using Docker Compose:

```bash
docker compose up -d
```

**Explanation:**

- **`up -d`**: Builds, (re)creates, starts, and attaches to containers in detached mode.


### Accessing Jellyfin

After successfully starting the services, access the Jellyfin web interface:

1. **Open a Web Browser:**
    
    Navigate to `http://localhost:8096` or `http://your_server_ip:8096`.
    
2. **Jellyfin Setup:**
    - **First-Time Setup:** Follow the on-screen instructions to set up your Jellyfin server.
    - **Media Libraries:** When adding libraries, point `/mnt/renterd`.

**Note:** Ensure that the directories specified in the `docker-compose.yml` exist and have appropriate permissions.

## Security Considerations

- **Environment Variables:**
    - Store sensitive information like `S3_ACCESS_KEY_ID` and `S3_SECRET_ACCESS_KEY` securely in the `.env`.
- **Network Exposure:**
    - Expose only necessary ports. If deploying in a production environment, consider using a reverse proxy with HTTPS.

## Troubleshooting

### Checking Logs

To inspect the logs of the Jellyfin container:

```bash
docker compose logs -f jellyfin
```

Press `Ctrl + C` to exit the log view.

## Updating the Container

To update your running `renterd-jellyfin` container with the latest changes:

1. **Rebuild the Docker Image:**

```bash
docker compose pull
```

1. **Redeploy the Container:**

```bash
docker compose up -d jellyfin
```

---

## License

This project is licensed under the [MIT License](./LICENSE).