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

## How it works

Each Ghostscript version gets its own Docker image. Older versions are compiled from source on a base image with a compatible libc; newer versions use distro packages directly.

## Versions

| Version  | Year | Source       | Base Image         |
|----------|------|--------------|---------------------|
| 8.63     | 2008 | Source build | ubuntu:18.04        |
| 8.64     | 2009 | Source build | ubuntu:18.04        |
| 8.71     | 2010 | Source build | ubuntu:18.04        |
| 9.01     | 2011 | Source build | ubuntu:18.04        |
| 9.06     | 2012 | Source build | ubuntu:18.04        |
| 9.10     | 2013 | Source build | ubuntu:22.04        |
| 9.14     | 2014 | Source build | ubuntu:22.04        |
| 9.18     | 2015 | Source build | ubuntu:22.04        |
| 9.20     | 2016 | Source build | ubuntu:22.04        |
| 9.22     | 2017 | Source build | ubuntu:22.04        |
| 9.50     | 2019 | apt package  | ubuntu:20.04        |
| 9.52     | 2020 | zypper pkg   | opensuse/leap:15.5  |
| 9.53.3   | 2020 | apt package  | debian:bullseye     |
| 9.55.0   | 2021 | apt package  | ubuntu:22.04        |
| 9.56.1   | 2022 | dnf package  | fedora:36           |
| 10.0.0   | 2022 | apt package  | debian:bookworm     |
| 10.02.1  | 2023 | apt package  | ubuntu:24.04        |
| 10.03.0  | 2024 | Source build | ubuntu:24.04        |
| 10.03.1  | 2024 | dnf package  | fedora:41           |
| 10.05.0  | 2025 | apt package  | ubuntu:25.04        |
| 10.05.1  | 2025 | apt package  | debian:trixie       |
| 10.07.1  | 2026 | Source build | ubuntu:24.04        |

## Adding a new version

1. Create a Dockerfile under `versions/<version>/Dockerfile`
2. Add the entry to the table above
3. Build: `docker build -t gs-<version> versions/<version>/`
4. Run: `docker run --rm -v $(pwd):/work gs-<version> gs <args>`
