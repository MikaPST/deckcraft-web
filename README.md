<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/deckcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.deckcraft.png" width="128" height="128" alt="DeckCraft icon">
</p>

<h1 align="center">DeckCraft Web — Docker Image</h1>

<p align="center">
  A self-hosted browser build of DeckCraft, an open-source presentation and slide designer.
  <br>
  Packaged as a lightweight static web application and published to GitHub Container Registry.
</p>

<p align="center">
  <a href="https://github.com/MikaPST/deckcraft-web/actions/workflows/publish-image.yml"><img src="https://github.com/MikaPST/deckcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish image"></a>
  <a href="https://ghcr.io/mikapst/deckcraft-web"><img src="https://img.shields.io/badge/GHCR-latest-blue?logo=docker" alt="GHCR"></a>
  <img src="https://img.shields.io/badge/Docker%20Compose-supported-blue?logo=docker" alt="Docker Compose">
</p>

---

## Why this repository?

This repository provides a Docker image for the official [DeckCraft](https://github.com/storytold/deckcraft) web application.

The goal is to make DeckCraft easy to deploy on a self-hosted server such as Unraid, without having to build the application manually.

- Build the web application directly from the official upstream repository.
- Automatically publish the resulting Docker image to GitHub Container Registry (GHCR).
- Rebuild periodically to follow upstream changes.
- Provide a simple Docker Compose configuration.
- Make the image suitable for Unraid and other Docker environments.
- Keep the Docker packaging separate from the upstream application.

This repository is a packaging project only. It is not a fork of DeckCraft and does not modify the upstream source code.

## How it works

DeckCraft is an open-source presentation and slide designer written in Rust. Its web version runs in the browser through WebAssembly and uses `eframe`/`wgpu`, with WebGPU and a WebGL2 fallback.

The application provides tools for creating and editing slide decks, including layouts, themes, shapes, text, pictures, tables, charts, transitions and animations. The upstream project is currently in alpha, so capabilities and compatibility may evolve.

The Docker image:

- Serves the prebuilt DeckCraft web application as static files using NGINX.
- Does not run a desktop session or remote desktop.
- Does not add a server-side presentation engine or processing backend.
- Does not provide server-side user accounts or centralized document storage.

The browser application handles its own interaction and rendering. Do not assume that presentations are automatically stored on the Docker host: persistence and file handling depend on the web application's current implementation and browser capabilities.

## Docker image

The image is published on GitHub Container Registry:

```text
ghcr.io/mikapst/deckcraft-web:latest
```

### Option 1 — Docker Compose (recommended)

Create a `docker-compose.yml` file:

```yaml
services:
  deckcraft-web:
    image: ghcr.io/mikapst/deckcraft-web:latest
    container_name: deckcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Start the container:

```bash
docker compose up -d
```

Open DeckCraft in your browser:

```text
http://YOUR_SERVER_IP:8080
```

#### Useful Compose commands

View container logs:

```bash
docker compose logs -f
```

Update the image:

```bash
docker compose pull
docker compose up -d
```

Stop and remove the container:

```bash
docker compose down
```

The container listens on port `8080`. No volume or environment variable is required by this static web image.

### Option 2 — Run directly with Docker

Pull the image:

```bash
docker pull ghcr.io/mikapst/deckcraft-web:latest
```

Run the container:

```bash
docker run -d \
  --name deckcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/deckcraft-web:latest
```

Then open:

```text
http://YOUR_SERVER_IP:8080
```

### Network access and reverse proxies

The container exposes an HTTP service on port `8080`.

You can place it behind a reverse proxy such as Traefik or NGINX Proxy Manager to access it through a domain name and HTTPS.

For public access, use HTTPS through a reverse proxy rather than exposing the unencrypted HTTP endpoint directly to the Internet.

DeckCraft uses browser graphics APIs. WebGPU generally requires a secure context (HTTPS, or localhost); WebGL2 is available as a fallback where supported by the browser and device.

## Automated builds

The GitHub Actions workflow builds the web application from the official `storytold/deckcraft` repository and publishes the resulting image to GHCR.

| Trigger | Behaviour |
|---|---|
| Weekly schedule | Checks for upstream changes and rebuilds the image |
| Manual execution | Allows a build to be started manually |
| Packaging changes | Rebuilds when the workflow, README or Compose file changes |

Published tags:

- `latest` — the most recent successful build.
- `upstream-<commit>` — the upstream commit used for that build.

Supported image architectures:

- `linux/amd64`
- `linux/arm64`

> **Important:** DeckCraft is under active development and is currently in alpha. The `latest` tag is mutable and may contain changes or regressions from upstream. For a stable deployment, consider pinning a specific image digest.

The workflow has been prepared but must be validated by its first execution in GitHub Actions. Until that run succeeds, the build and publication process should be considered unverified.

## Unraid Community Applications

This image can be used with Unraid through Docker Compose or a Community Applications template.

The Unraid template is maintained separately in the [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps) repository.

This repository contains the Docker image build and publication workflow; the Community Applications repository contains the Unraid-specific template.

## Upstream project

**DeckCraft** is an open-source presentation and slide designer written in Rust.

It aims to provide a modern slide-authoring experience, including:

- Slide masters, layouts, themes and sections.
- Shapes, connectors, text formatting and hyperlinks.
- Pictures, tables, charts and SmartArt basics.
- Transitions, animations and full-screen presentations.
- Import and export of `.pptx` files, plus PDF and image exports in the upstream application.

The upstream project documents its browser build as a static WebAssembly site. Not every desktop capability should be assumed to be available or identical in the browser build.

- **Source code:** [github.com/storytold/deckcraft](https://github.com/storytold/deckcraft)
- **Official website:** [getartcraft.com/apps/deckcraft](https://getartcraft.com/apps/deckcraft)
- **Releases:** [GitHub Releases](https://github.com/storytold/deckcraft/releases)

This repository packages the upstream web application for Docker. It does not include the native desktop application, CLI or MCP server.

## License and attribution

DeckCraft is dual-licensed under **MIT** or **Apache-2.0**, at your option. Copyright and required notices are maintained by the upstream project in its `NOTICE` file.

Bundled fonts, icons, images and other assets may have their own licenses. Their authors, sources and licenses are documented in the upstream `ATTRIBUTION.md`.

The DeckCraft and ArtCraft names, logos and related brand assets are trademarks of the upstream project and are not covered by the software license.

The Docker packaging files in this repository are provided separately from the upstream application. Refer to the upstream repository for the complete license texts, notices and asset attributions.

## Disclaimer

This is an independent community packaging project and is not affiliated with, endorsed by, or supported by the ArtCraft team.

All product names, trademarks and logos are the property of their respective owners.
