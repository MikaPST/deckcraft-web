<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/deckcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.deckcraft.png" width="128" height="128" alt="DeckCraft application icon">
</p>

<h1 align="center">DeckCraft Web — Docker Image</h1>
<p align="center">
  A community-maintained Docker image for
  <a href="https://github.com/storytold/deckcraft">DeckCraft</a>,
  providing its browser-based presentation editor through a self-hosted web interface.
  <br>
  Build automation and image publication to GitHub Container Registry (GHCR).
</p>

<p align="center">
  <a href="https://github.com/MikaPST/deckcraft-web/actions/workflows/publish-image.yml">
    <img src="https://github.com/MikaPST/deckcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish DeckCraft Web">
  </a>
  <a href="https://github.com/MikaPST/deckcraft-web/releases/latest">
    <img src="https://img.shields.io/github/v/release/MikaPST/deckcraft-web?label=Release&logo=github&color=blue" alt="Latest Release">
  </a>
  <a href="https://github.com/MikaPST/deckcraft-web/pkgs/container/deckcraft-web">
    <img src="https://ghcr-badge.egpl.dev/mikapst/deckcraft-web/latest_tag?trim=major&label=GHCR" alt="GHCR image tag">
  </a>
  <a href="docker-compose.yml">
    <img src="https://img.shields.io/badge/Docker%20Compose-ready-2496ED?logo=docker&logoColor=white" alt="Docker Compose configuration">
  </a>
</p>

---

## Why this repository?

The official DeckCraft project includes a WebAssembly version that can be hosted as a static website. This repository packages it into a Docker image for straightforward self-hosting.

The goals are to:

- Build DeckCraft Web from the **official upstream release source**, not an unrelated branch head.
- Publish ready-to-run Docker images to GHCR for `linux/amd64` and `linux/arm64`.
- Detect newly published official versions automatically and publish matching, versioned images.
- Create a matching GitHub Release with the official release notes **after** successful image publication.
- Provide a deployment option for Unraid Community Applications.
- Include a ready-to-use Docker Compose file.
- Maintain Docker, NGINX and build automation independently of the upstream application.
- Keep all packaging files version-controlled rather than generating Docker and NGINX configurations inside a workflow.

This is a packaging and distribution project, not a fork intended to develop a separate presentation editor.

## How it works

DeckCraft is an open-source presentation and slide editor written in Rust. Its web version runs in the browser using WebAssembly, with WebGPU rendering and a WebGL2 fallback when supported.

- **Browser-based interface:** create and edit slides directly from a browser.
- **Presentation tools:** layouts, themes, text, shapes, images, charts, tables, transitions and animations; exact feature availability follows the upstream release.
- **File operations:** the web application handles opening, saving and exporting presentations in the browser, without a server-side document service.
- **No remote desktop:** no VNC, Linux desktop session or dedicated backend is required.
- **Self-hosted:** serve the editor from your own Unraid or Docker host, optionally behind a secure reverse proxy.

DeckCraft supports `.pptx` and its own `.deckcraft` presentation format, and has a browser-based host integration contract (`?host=parent`) for same-origin embedding. The container only delivers static assets. It does not automatically retain users' slides on the server.

## Docker image

The image is published to GitHub Container Registry:

```text
ghcr.io/mikapst/deckcraft-web:latest
```

Each packaged official release is also published under its exact upstream version tag, for example:

```text
ghcr.io/mikapst/deckcraft-web:v0.19.0
```

### Option 1 — Docker Compose (recommended)

A `docker-compose.yml` file is included at the repository root:

```yaml
services:
  deckcraft-web:
    image: ghcr.io/mikapst/deckcraft-web:latest
    container_name: deckcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Start DeckCraft Web from the repository root:

```bash
docker compose up -d
```

Open the interface:

```text
http://YOUR_SERVER_IP:8080
```

Replace `YOUR_SERVER_IP` with your server's address. On the Docker host itself, `http://localhost:8080` is also suitable.

#### Useful Compose commands

```bash
# Inspect logs
docker compose logs -f

# Pull updates and recreate the container
docker compose pull
docker compose up -d

# Stop and remove the container
docker compose down
```

This basic setup requires no `.env` file, database or persistent Docker volume. To select another host port, change the left side of the mapping, for example `8091:8080`.

### Option 2 — Run directly with Docker

```bash
docker pull ghcr.io/mikapst/deckcraft-web:latest

docker run -d \
  --name deckcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/deckcraft-web:latest
```

Open the application at `http://YOUR_SERVER_IP:8080`.

### Network access and reverse proxies

The container listens on port `8080` over HTTP. For public access, terminate HTTPS using a reverse proxy such as Traefik or NGINX Proxy Manager and restrict access as appropriate.

DeckCraft uses WebGPU when available, which generally requires **HTTPS or localhost**. WebGL2 is the browser fallback. The upstream web hosting guide does not require special SharedArrayBuffer isolation headers.

NGINX serves `.wasm` with `application/wasm`, compresses static assets, revalidates `index.html` and caches content-hashed JavaScript and WebAssembly assets. If you embed DeckCraft in an iframe, do not add `X-Frame-Options: DENY` or a `frame-ancestors` policy that forbids your embedding origin. The upstream host bridge (`?host=parent`) is restricted to the same origin.

## Repository structure

This repository maintains its own Docker packaging while retrieving the official DeckCraft release source during the build.

```text
deckcraft-web/
├── .github/
│   └── workflows/
│       └── publish-image.yml
├── .dockerignore
├── Dockerfile
├── nginx.conf
├── docker-compose.yml
└── README.md
```

| File | Purpose |
|---|---|
| `Dockerfile` | Builds the NGINX runtime image from prepared static files, includes upstream notices and provides a health check. |
| `nginx.conf` | Configures WebAssembly MIME handling, caching, gzip, security headers and `/healthz`. |
| `.dockerignore` | Keeps the Docker build context limited to the generated site and required legal notices. |
| `docker-compose.yml` | Defines the standard Docker Compose deployment. |
| `.github/workflows/publish-image.yml` | Detects releases, builds the WebAssembly site, publishes multi-platform GHCR images and creates GitHub Releases. |
| `README.md` | Documents the application, deployment, packaging and attribution. |

### Docker build architecture

The image uses **official DeckCraft source code** and separate Docker packaging maintained in this repository. The official browser build is based on **Rust, WebAssembly and Trunk**.

The build process follows these steps:

1. GitHub Actions identifies the latest published DeckCraft release or the requested official tag.
2. The workflow checks out that exact release under `upstream/`.
3. It reads the `craft-fonts-ref` pinned in the selected upstream release's `.github/workflows/release.yml` and checks out that exact `storytold/craft-fonts` commit.
4. It installs Rust `1.96.1`, the `wasm32-unknown-unknown` target and Trunk.
5. It builds the application with `trunk build --release` from `upstream/apps/deckcraft-web`. The official Trunk configuration places files under `upstream/dist/web/` with relative URLs.
6. The workflow validates `index.html`, generated `.js` and `.wasm` files, and relative asset URLs.
7. It assembles the web files, repository-managed `Dockerfile`, `nginx.conf`, `.dockerignore`, and the available application and font license notices into a temporary image context.
8. Docker Buildx publishes the runtime image to GHCR for `linux/amd64` and `linux/arm64`.
9. Only after the successful image push does the workflow create or update the matching GitHub Release.

The application is compiled once into architecture-independent browser assets, then packaged for both supported container architectures. The NGINX image runs as a **non-root user**, listens on port `8080` and exposes `/healthz` with a Docker health check.

Unlike WordCraft, DeckCraft's checked release build does not require a separate `craft-i18n` checkout in this packaging workflow. The required `craft-fonts` revision is sourced from the upstream release itself to avoid accidentally embedding assets from a floating `main` branch.

> [!NOTE]
> New upstream releases may change Rust requirements, font manifests or build output structure. The GitHub Actions build is the authoritative integration test for each official version.

## Automated builds

GitHub Actions monitors the official DeckCraft releases, and packaging changes on this repository trigger rebuilding the latest upstream version.

| Trigger | Behaviour |
|---|---|
| Daily schedule | Checks the latest published official DeckCraft release and builds it if it is not published here |
| Manual workflow dispatch | Builds the latest official release, or a selected `upstream_tag` |
| Force rebuild | Rebuilds an already published version and updates its matching GitHub Release |
| Packaging changes on `main` | Rebuilds the latest upstream release when the README, workflow, Dockerfile, NGINX, `.dockerignore` or Compose configuration changes |

### Published tags

Every successful build publishes these image tags:

- `latest` — the most recently packaged official release (mutable).
- `<upstream-version>` — the exact upstream tag, such as `v0.19.0`.
- `upstream-<commit>` — the first 12 characters of the upstream source commit.

The image is built for `linux/amd64` and `linux/arm64`. Its contents are derived from the official upstream tag and the independent packaging files in this repository.

### GitHub Releases

After Docker Buildx successfully publishes the multi-platform image to GHCR, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/deckcraft-web/releases) using the corresponding upstream tag.

Each published release contains:

- A link to the official upstream release and its notes.
- The exact upstream release tag and source commit.
- The `craft-fonts` commit used during the build.
- The published Docker tags (`latest`, the upstream version and `upstream-<commit>`).
- Supported architectures: `linux/amd64` and `linux/arm64`.
- The Docker manifest digest, when available.
- A notice that this is community-maintained packaging, not an official ArtCraft distribution.

The Release badge at the top reflects the latest version published **in this repository**.

Scheduled and ordinary manual checks skip versions already released here. To rebuild, open **Actions → Build and publish DeckCraft Web → Run workflow** and enable `force_rebuild`. Packaging changes on `main` rebuild the latest upstream version automatically.

> [!IMPORTANT]
> A GitHub Release is created or updated **only after** a successful GHCR push. The `latest` tag is mutable, and forcing a rebuild may also replace versioned tags. Pin a digest if exact reproducibility is required.

DeckCraft is actively developed. Test a new image before depending on it for important presentations.

## Unraid Community Applications

DeckCraft Web is available through the maintainer's separate Unraid Community Applications template repository:

- **Unraid Community Applications repository:** [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps)
- **DeckCraft Web template:** [deckcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/deckcraft-web.xml)
- **Container port:** `8080` (the current Unraid template defaults to host port `8091`)

The template references `ghcr.io/mikapst/deckcraft-web:latest` and is maintained separately from this packaging project.

## Upstream project

DeckCraft is an open-source presentation and slide editor developed by the ArtCraft team and contributors.

- **Source code:** [storytold/deckcraft](https://github.com/storytold/deckcraft)
- **Official website:** [getartcraft.com/apps/deckcraft](https://getartcraft.com/apps/deckcraft)
- **Official web hosting guide:** [Hosting DeckCraft for the web](https://github.com/storytold/deckcraft/blob/main/packaging/web/README.md)
- **Official releases:** [DeckCraft releases](https://github.com/storytold/deckcraft/releases)

The MikaPST GitHub Releases reflect independently maintained Docker packaging, not official upstream distributions.

## License and attribution

DeckCraft is dual-licensed under **MIT or Apache-2.0**, at the user's option. Copyright remains with the upstream authors and contributors.

The upstream project includes licenses, notices and third-party asset attribution. The Docker image copies available `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`, and `ATTRIBUTION.md` files, plus the `OFL.txt` font licenses found in the pinned `craft-fonts` checkout, into `/usr/share/licenses/deckcraft/`.

- [MIT License](https://github.com/storytold/deckcraft/blob/main/LICENSE-MIT)
- [Apache 2.0 License](https://github.com/storytold/deckcraft/blob/main/LICENSE-APACHE)
- [NOTICE](https://github.com/storytold/deckcraft/blob/main/NOTICE)
- [Asset attribution](https://github.com/storytold/deckcraft/blob/main/ATTRIBUTION.md)

The upstream application's licenses do not automatically set the license for this repository's independently maintained packaging configuration. Refer to a repository `LICENSE` file if one is published.

The DeckCraft and ArtCraft names, branding and logos retain their own trademark and usage terms. This repository is independently maintained and **is not affiliated with or endorsed by** the original project maintainers.

## Disclaimer

DeckCraft is actively developed and its behaviour, file compatibility and supported features may change between releases. Use the application and keep independent backups of important presentation files.
