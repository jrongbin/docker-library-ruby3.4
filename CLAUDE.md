# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Docker image definitions for Ruby 3.4 with Node.js 22 support. The current images are based on Debian Bullseye and Debian Bookworm. They are designed as development environments that include:

- Ruby 3.4 runtime (base image)
- Node.js 22.x
- OpenResty (nginx-based web server)
- pnpm or Yarn package manager, depending on the Dockerfile variant
- System services managed by s6-overlay (SSH, cron, OpenResty)
- Common development tools (git, vim, curl, wget, etc.)

## Architecture

Current Dockerfiles:

- `Dockerfile.de11-no22-op-s631-ya`: Debian 11 / Bullseye, Node.js 22.x, OpenResty, s6-overlay 3.1.x, Yarn
- `Dockerfile.de12-no22-op-pn-s631`: Debian 12 / Bookworm, Node.js 22.x, OpenResty, pnpm, s6-overlay 3.1.x

Each Dockerfile:

1. Uses a Ruby 3.4 Debian base image
2. Installs system dependencies and tools
3. Configures locale to en_US.UTF-8
4. Sets up s6-overlay to manage multiple services
5. Installs OpenResty web server with architecture-specific packages
6. Adds Node.js 22.x from NodeSource repository
7. Exposes ports 22 (SSH) and 80 (HTTP)
8. Starts s6-overlay as the main process

## Dockerfile Naming

Dockerfile names use compact capability tokens in alphabetical order:

```text
Dockerfile.<token>-<token>-<token>
```

Use lowercase tokens, separate tokens with `-`, and keep version numbers to major or major/minor precision when possible. Ruby 3.4 is the repository baseline and does not need to be encoded unless multiple Ruby versions are introduced.

Current token meanings:

- `de11`: Debian 11 / Bullseye
- `de12`: Debian 12 / Bookworm
- `no22`: Node.js 22.x
- `op`: OpenResty
- `pn`: pnpm
- `s631`: s6-overlay 3.1.x
- `ya`: Yarn

Example for the current Bookworm image:

```text
Dockerfile.de12-no22-op-pn-s631
```

The Debian token should be included when the Debian major version is known. If a generator only sees a base tag such as `ruby:3.4-bookworm` and does not map codenames to Debian major versions, it may initially omit the `de` token; add the matching token manually once the mapping is known.

## Docker Image Tags

Publish both a full token tag and a friendly alias:

- Full token tag: remove the `Dockerfile.` prefix, for example `de12-no22-op-pn-s631`
- Friendly alias: use the runtime-first Docker style, for example `node22-bookworm` or `node22-bullseye`

## Docker Commands

```bash
# Build the Bookworm image
docker build -f Dockerfile.de12-no22-op-pn-s631 \
  -t ruby-node22:de12-no22-op-pn-s631 \
  -t ruby-node22:node22-bookworm .

# Build the Bullseye image
docker build -f Dockerfile.de11-no22-op-s631-ya \
  -t ruby-node22:de11-no22-op-s631-ya \
  -t ruby-node22:node22-bullseye .

# Run the container
docker run -d -p 22:22 -p 80:80 ruby-node22:node22-bookworm

# Interactive shell
docker run -it ruby-node22:node22-bookworm bash
```

## Key Configuration Details

- **Base Images**: ruby:3.4-bullseye and ruby:3.4-bookworm
- **Services**: SSH server, cron, OpenResty web server (managed by s6-overlay)
- **Locale**: en_US.UTF-8
- **Working Directory**: /opt
- **Exposed Ports**: 22 (SSH), 80 (HTTP)
- **Architecture Support**: Both x86_64/amd64 and aarch64/arm64 for OpenResty packages

## Important Notes

- The image automatically starts s6-overlay which manages SSH, cron, and OpenResty services
- OpenResty packages are installed from different repositories based on CPU architecture
- All system packages are cleaned up after installation to reduce image size

## Version-Specific Files

Keep shared service definitions in top-level directories such as `s6-services/` only while all Dockerfile variants can use exactly the same files. If a Debian version or image variant needs different service scripts, follow the Docker Official Images style and move variant-specific files next to that Dockerfile under a versioned directory, for example:

```text
de11-no22-op-s631-ya/
  Dockerfile
  s6-services/
de12-no22-op-pn-s631/
  Dockerfile
  s6-services/
```

In that layout, each Dockerfile should copy from its own local context paths, so one variant can change without accidentally affecting another.
