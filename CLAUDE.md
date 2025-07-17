# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a Docker image definition for Ruby 3.4 with Node.js 22 support, based on Debian Bullseye. The image is designed as a development environment that includes:

- Ruby 3.4 runtime (base image)
- Node.js 22.x
- OpenResty (nginx-based web server)
- System services managed by supervisor (SSH, cron, OpenResty)
- Common development tools (git, vim, curl, wget, etc.)

## Architecture

The project consists of a single Dockerfile (`Dockerfile.node22-bullseye`) that:

1. Uses `ruby:3.4-bullseye` as the base image
2. Installs system dependencies and tools
3. Configures locale to en_US.UTF-8  
4. Sets up supervisor to manage multiple services
5. Installs OpenResty web server with architecture-specific packages
6. Adds Node.js 22.x from NodeSource repository
7. Exposes ports 22 (SSH) and 80 (HTTP)
8. Starts supervisor as the main process

## Docker Commands

```bash
# Build the image
docker build -f Dockerfile.node22-bullseye -t ruby-node22 .

# Run the container
docker run -d -p 22:22 -p 80:80 ruby-node22

# Interactive shell
docker run -it ruby-node22 bash
```

## Key Configuration Details

- **Base Image**: ruby:3.4-bullseye
- **Services**: SSH server, cron, OpenResty web server (managed by supervisor)
- **Locale**: en_US.UTF-8
- **Working Directory**: /opt
- **Exposed Ports**: 22 (SSH), 80 (HTTP)
- **Architecture Support**: Both x86_64/amd64 and aarch64/arm64 for OpenResty packages

## Important Notes

- The image automatically starts supervisor which manages SSH, cron, and OpenResty services
- Environment variables are written to `/etc/environment` on container startup
- OpenResty packages are installed from different repositories based on CPU architecture
- All system packages are cleaned up after installation to reduce image size