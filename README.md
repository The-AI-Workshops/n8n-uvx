## Repository Description

Self-hosted N8N integration for MCP UVX servers via Docker—run UVX-powered MCP servers locally in your N8N workflows without any cloud dependency.

---

# n8n-uvx

&#x20;

> A Dockerized setup that integrates Astral UV/UVX MCP servers directly into your local N8N instance—no external cloud services required.

## Table of Contents

* [Introduction](#introduction)
* [Features](#features)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
* [Dockerfile Overview](#dockerfile-overview)
* [Environment Variables (.env)](#environment-variables-env)
* [Docker Compose Setup](#docker-compose-setup)
* [Configuration](#configuration)
* [Usage](#usage)
* [Additional Resources](#additional-resources)
* [Contributing](#contributing)
* [License](#license)

## Introduction

This repository provides everything you need to self-host MCP UVX servers inside your N8N workflows using Docker. It installs the Astral UV/UVX CLI tools into an N8N container, exposes a data directory for MCP assets, and configures a Docker Compose service for seamless local automation.

## Features

* 🐳 **Docker-First**: Fully containerized setup for portability and isolation.
* ⚙️ **Astral CLI Tools**: Installs `uv` and `uvx` via Astral’s installer for MCP server management.
* 📁 **Persistent Data Volumes**: Separate directories for N8N workflows and MCP assets.
* 🔒 **Local-Only**: Keep all data and execution on your machine—ideal for sensitive or offline environments.

## Prerequisites

* [Docker](https://docs.docker.com/get-docker/) (20.10+)
* [Docker Compose](https://docs.docker.com/compose/install/)
* A basic local N8N setup (v0.x or later)
* Familiarity with Docker and environment variables

## Installation

1. **Clone this repository**

   ```bash
   git clone https://github.com/The-AI-Workshops/n8n-uvx.git
   cd n8n-uvx
   ```
2. **Copy ****`.env.example`**** to ****`.env`**** and edit it** (update variables as needed — see [Environment Variables](#environment-variables-env)).
3. **Build the Docker image**:

   ```bash
   docker build -t n8n-uvx .
   ```
4. **Launch with Docker Compose**:

   ```bash
   docker-compose up -d
   ```

\*\*Alternative with \*\***`docker run`** (no Compose):

```bash
docker run -d \
  --name n8n-uvx \
  -p ${N8N_PORT}:5678 \
  -e N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE} \
  -e N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER} \
  -e N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD} \
  -e N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=${N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE} \
  -v ${N8N_DATA_DIR}:/home/node/.n8n \
  -v ${MCP_DATA_DIR}:/data/mcp \
  n8n-uvx
```

## Dockerfile Overview

The `Dockerfile` extends the official N8N image and includes steps to install system dependencies and Astral UV/UVX:

```dockerfile
FROM n8nio/n8n:latest

USER root

# Install necessary system packages
RUN apk add --no-cache \
    curl \
    git \
    build-base \
    chromium \
    bash \
    tar \
    xz \
    util-linux \
    coreutils

# Install Astral uv/uvx and make available system-wide
RUN curl -Ls https://astral.sh/uv/install.sh | bash \
    && cp /root/.local/bin/uv /usr/local/bin/uv \
    && cp /root/.local/bin/uvx /usr/local/bin/uvx \
    && chmod a+rx /usr/local/bin/uv /usr/local/bin/uvx \
    && mkdir -p /data/mcp \
    && chown -R node:node /data/mcp

ENV PATH="/usr/local/bin:/root/.local/bin:${PATH}"

USER node
```

## Environment Variables (.env)

Create a `.env` at the project root with these values:

```dotenv
# N8N Settings
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=admin123
N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
N8N_PORT=5678

# Data Volume Paths
N8N_DATA_DIR=./data
MCP_DATA_DIR=./mcp
```

## Docker Compose Setup

Use explicit variable mappings in `docker-compose.yml`:

```yaml
version: '3.8'

services:
  n8n:
    build: .
    ports:
      - "${N8N_PORT}:5678"
    environment:
      # N8N Core
      N8N_BASIC_AUTH_ACTIVE:           "${N8N_BASIC_AUTH_ACTIVE}"
      N8N_BASIC_AUTH_USER:             "${N8N_BASIC_AUTH_USER}"
      N8N_BASIC_AUTH_PASSWORD:         "${N8N_BASIC_AUTH_PASSWORD}"
      N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE: "${N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE}"

      # Volume Paths (for clarity)
      N8N_DATA_DIR: "${N8N_DATA_DIR}"
      MCP_DATA_DIR: "${MCP_DATA_DIR}"
    volumes:
      - "${N8N_DATA_DIR}:/home/node/.n8n"
      - "${MCP_DATA_DIR}:/data/mcp"
```

## Configuration

* **Ports**: Adjust `N8N_PORT` if port 5678 is in use.
* **Volumes**: Ensure `./data` and `./mcp` exist with proper permissions.
* **Authentication**: Change default `admin123` password to a secure secret.

## Usage

1. Visit `http://localhost:${N8N_PORT}` and log in.
2. Install the **MCP Client** node if needed: **Settings > Community Nodes > n8n-nodes-mcp-client**.
3. Create workflows using the **uv**/**uvx** CLI commands via N8N’s **Execute Command** node or custom nodes.

## Additional Resources

* **Astral UV/UVX CLI**: [https://astral.sh/docs/uv](https://astral.sh/docs/uv)
* **Awesome MCP Servers**: [https://github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
* **N8N Docker Docs**: [https://docs.n8n.io/getting-started/docker/](https://docs.n8n.io/getting-started/docker/)

## Contributing

1. Fork the repo.
2. Create a branch (`git checkout -b feature/xyz`).
3. Commit changes (`git commit -m "Add feature"`).
4. Push to origin and open a PR.

## License

This project is licensed under the [MIT License](LICENSE).
