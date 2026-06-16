# ghostscript_version_graveyard

A collection of Ghostscript binaries spanning different versions and years, each packaged in its own Docker container so they can all run on a single host without conflicts.

## Why

Ghostscript behavior varies across versions — rendering quirks, security patches, PostScript/PDF interpreter changes. Having a graveyard of old versions lets you:

- Reproduce bugs tied to a specific Ghostscript release
- Compare output across versions (regression testing, fuzz testing)
- Test how a document renders on the same Ghostscript version a particular distro ships

## Getting started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed and running

### Setup

```bash
git clone https://github.com/w0ot-net/ghostscript_version_graveyard.git
cd ghostscript_version_graveyard
```

### Usage

Use the `gs-run` wrapper to invoke any version. The Docker image is built automatically on first run (source builds may take a few minutes; package-based versions are faster).

```bash
# Check a version
./gs-run 9.50 -- --version

# Check a companion debug build
./gs-run --debug 9.50 -- --version

# Check the combined image's normal and debug binaries
./gs-run --combined 9.50 -- --version
./gs-run --combined-debug 9.50 -- --version

# List available versions
./gs-run

# Convert PostScript to PDF
./gs-run 9.50 -- -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=out.pdf input.ps

# Render a PDF to PNG
./gs-run 10.02.1 -- -dBATCH -dNOPAUSE -sDEVICE=png16m -r300 -sOutputFile=page-%d.png input.pdf

# Compare output across versions
for v in 9.50 10.02.1 10.07.1; do
  ./gs-run $v -- -dBATCH -dNOPAUSE -sDEVICE=png16m -sOutputFile="out-$v.png" input.pdf
done
```

Files in your current directory are mounted at `/work` inside the container, so input/output paths are relative to where you run the command.

### Pre-building all images

To build every version up front instead of on first use:

```bash
for dir in versions/*/; do
  version=$(basename "$dir")
  echo "Building gs-$version..."
  docker build -t "gs-$version" "$dir"
done
```

### Pulling published images

Images can be published to GitHub Container Registry as `ghcr.io/w0ot-net/gs-<version>:latest`.
Companion debug builds are published as `ghcr.io/w0ot-net/gs-<version>:debug`.
Combined builds are published as `ghcr.io/w0ot-net/gs-<version>:combined`.
The `latest` tag remains the normal runtime build; debug and combined builds are additional tags.

```bash
# Pull a published image
docker pull ghcr.io/w0ot-net/gs-9.50:latest

# Pull the companion debug image
docker pull ghcr.io/w0ot-net/gs-9.50:debug

# Pull the combined image with both gs and gs-debug
docker pull ghcr.io/w0ot-net/gs-9.50:combined

# Check a published image
docker run --rm ghcr.io/w0ot-net/gs-9.50:latest --version

# Run Ghostscript from the current directory
docker run --rm -v "$(pwd):/work" ghcr.io/w0ot-net/gs-9.50:latest \
  -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=out.pdf input.ps
```

## How it works

Each Ghostscript version gets its own Docker image. Older versions are compiled from source on a base image with a compatible libc; newer versions use distro packages directly.

Images keep the canonical `gs-<version>` tag. Each Dockerfile also labels the image with the architecture of the hosted `gs` executable:

- `org.opencontainers.image.architecture=amd64`
- `net.w0ot.ghostscript.cpu-architecture=x86_64`

Inspect those labels with:

```bash
docker image inspect gs-9.50 \
  --format '{{ index .Config.Labels "net.w0ot.ghostscript.cpu-architecture" }}'
```

### Debug Builds

Debug images are companions to the normal images. They do not replace the default `gs-<version>:latest` images.

Locally, build or run a debug companion with:

```bash
scripts/build-debug-image 9.50
./gs-run --debug 9.50 -- --version
```

Debug companions are tagged as `gs-<version>:debug` and are built from public upstream Ghostscript source tarballs with `-g3 -O0 -fno-omit-frame-pointer`. They include `gdb`, `binutils`, and an unstripped `gs` executable with a `.debug_info` section.

Debug images add these labels:

- `net.w0ot.ghostscript.build-flavor=debug`
- `net.w0ot.ghostscript.debug-symbols=true`

### Combined Builds

Combined images are a third image set. They do not replace either the normal `latest` images or the companion `debug` images.

Locally, build or run a combined image with:

```bash
scripts/build-combined-image 9.50
./gs-run --combined 9.50 -- --version
./gs-run --combined-debug 9.50 -- --version
```

Combined images are tagged as `gs-<version>:combined` and contain:

- `gs`: the normal binary from `versions/<version>/Dockerfile`, preserving distro package builds whenever that Dockerfile uses one
- `gs-debug`: a source-built debug binary installed at `/usr/local/bin/gs-debug` with `-g3 -O0 -fno-omit-frame-pointer`

The combined image is assembled from the normal and debug companion images, so `gs-debug` is the same debug build published as `gs-<version>:debug`.

The combined build scripts default to at most two parallel compiler jobs, and the legacy autoconf builds default to one. Set `GS_BUILD_JOBS` to override that locally.

Combined images add this label:

- `net.w0ot.ghostscript.build-flavor=combined`

### Publishing to GHCR

The `Publish Ghostscript Images` GitHub Actions workflow publishes images to GitHub Container Registry. Run it manually from the Actions tab with either:

- a single version, such as `9.50`
- `all` to build, verify, and publish every version under `versions/`

For each image, the workflow:

- builds `versions/<version>/Dockerfile`
- verifies `gs --version`
- verifies the Docker platform architecture is `amd64`
- verifies the architecture labels are present
- pushes `ghcr.io/w0ot-net/gs-<version>:latest`
- builds the companion debug image from source
- verifies the debug image has the same `gs --version`
- verifies the debug image has a `.debug_info` section
- pushes `ghcr.io/w0ot-net/gs-<version>:debug`
- builds the combined image with both `gs` and `gs-debug`
- verifies both commands report the expected version
- verifies `gs-debug` has a `.debug_info` section
- pushes `ghcr.io/w0ot-net/gs-<version>:combined`

The workflow uses the repository `GITHUB_TOKEN` with `packages: write`; no personal token or registry secret is required. After the first publish, set the package visibility to public in GitHub Packages if anonymous pulls should work.

## PostScript integer width

Ghostscript versions before 9.07 store PostScript integer objects as 32-bit C `int` values. In those builds, integer arithmetic that exceeds the 32-bit range is promoted to `realtype`; for example, `2147483647 1 add` produces a real value instead of an integer.

Ghostscript 9.07 introduced 64-bit PostScript integer objects by default. The versions in this repository therefore split as follows: 8.63 through 9.06 are 32-bit-only for PostScript integers, and 9.10 and later support 64-bit PostScript integers. This repository does not currently include 9.07, 9.08, or 9.09.

## Versions

| Version  | Year | Source       | Base Image         | 64-bit PS integers | `gs` CPU arch |
|----------|------|--------------|--------------------|--------------------|---------------|
| 8.63     | 2008 | Source build | ubuntu:18.04       | No                 | x86_64        |
| 8.64     | 2009 | Source build | ubuntu:18.04       | No                 | x86_64        |
| 8.71     | 2010 | Source build | ubuntu:18.04       | No                 | x86_64        |
| 9.01     | 2011 | Source build | ubuntu:18.04       | No                 | x86_64        |
| 9.06     | 2012 | apt package  | debian:jessie      | No                 | x86_64        |
| 9.10     | 2013 | yum package  | fedora:20          | Yes                | x86_64        |
| 9.14     | 2014 | yum package  | fedora:21          | Yes                | x86_64        |
| 9.18     | 2015 | apt package  | ubuntu:16.04       | Yes                | x86_64        |
| 9.20     | 2016 | dnf package  | fedora:25          | Yes                | x86_64        |
| 9.22     | 2017 | apt package  | ubuntu:18.04       | Yes                | x86_64        |
| 9.26     | 2018 | dnf package  | fedora:30          | Yes                | x86_64        |
| 9.50     | 2019 | apt package  | ubuntu:20.04       | Yes                | x86_64        |
| 9.52     | 2020 | zypper pkg   | opensuse/leap:15.5 | Yes                | x86_64        |
| 9.53.3   | 2020 | apt package  | debian:bullseye    | Yes                | x86_64        |
| 9.55.0   | 2021 | apt package  | ubuntu:22.04       | Yes                | x86_64        |
| 9.56.1   | 2022 | dnf package  | fedora:36          | Yes                | x86_64        |
| 10.0.0   | 2022 | apt package  | debian:bookworm    | Yes                | x86_64        |
| 10.02.1  | 2023 | apt package  | ubuntu:24.04       | Yes                | x86_64        |
| 10.03.0  | 2024 | Source build | ubuntu:24.04       | Yes                | x86_64        |
| 10.03.1  | 2024 | dnf package  | fedora:41          | Yes                | x86_64        |
| 10.05.0  | 2025 | apt package  | ubuntu:25.04       | Yes                | x86_64        |
| 10.05.1  | 2025 | apt package  | debian:trixie      | Yes                | x86_64        |
| 10.07.1  | 2026 | Source build | ubuntu:24.04       | Yes                | x86_64        |

## Adding a new version

1. Create a Dockerfile under `versions/<version>/Dockerfile`
2. Add the entry to the table above
3. Build: `docker build -t gs-<version> versions/<version>/`
4. Run: `docker run --rm -v $(pwd):/work gs-<version> gs <args>`
