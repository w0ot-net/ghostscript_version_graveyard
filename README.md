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

Each Ghostscript version gets its own Docker image. Some versions are compiled from public upstream source tarballs on a compatible base image; others use distro packages directly.

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

Debug companions are tagged as `gs-<version>:debug`. They include `gdb`,
`binutils`, an unstripped `gs` executable with a `.debug_info` section, and the
matching Ghostscript source tree under `/usr/src/ghostscript`.

`scripts/build-debug-image` prefers the distro's own debug symbols and only
falls back to a source build when no usable gs debug package exists:

- **Distro debug symbols (preferred).** For distro-package versions, the stripped
  `gs` (and `libgs.so`) are recombined with the distro debug-symbol package via
  `eu-unstrip`, so the executables carry a real `.debug_info` section matching the
  shipped binary. Symbols come from `ghostscript-dbg` / `*-dbgsym` on
  Debian/Ubuntu, `ghostscript-debuginfo` on Fedora/openSUSE/CentOS.
- **Source build (fallback).** Where the distro ships no usable gs debug package
  — the pure-source versions, and distro versions whose debug repo is gone or
  unusable (Debian jessie's path-style `-dbg`; Fedora 20/21/25 archive `elfutils`
  breakage; Debian bullseye/bookworm/trixie and Ubuntu plucky, which have no gs
  dbgsym; Ubuntu jammy/noble, whose ddebs only carries the GA-revision dbgsym, not
  the installed security point release) — `gs` is compiled from the matching
  upstream/orig tarball with `-g3 -O0 -fno-omit-frame-pointer`.

Either way the result satisfies the same checks: correct `gs --version`, a
`.debug_info` section in the `gs` executable, the source under
`/usr/src/ghostscript`, and the architecture/flavor labels.

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
- `gs-debug`: the debug binary from the companion debug image, installed at `/usr/local/bin/gs-debug` (distro debug symbols where available, otherwise a source build — see Debug Builds above)
- the Ghostscript source under `/usr/src/ghostscript`, inherited from the debug image

The combined image is assembled from the normal and debug companion images, so `gs-debug` is the same debug build published as `gs-<version>:debug`.

The combined build scripts default to at most two parallel compiler jobs, and the legacy autoconf builds default to one. Set `GS_BUILD_JOBS` to override that locally.

Combined images add this label:

- `net.w0ot.ghostscript.build-flavor=combined`

### Publishing to GHCR

The `Publish Ghostscript Images` GitHub Actions workflow builds, verifies, and
pushes all three flavors (`:latest`, `:debug`, `:combined`) for one version or
for `all` of them to `ghcr.io/w0ot-net/gs-<version>`. Run it manually from the
Actions tab. It uses the repository `GITHUB_TOKEN` with `packages: write`; no
personal token or registry secret is required. After the first publish, set the
package visibility to public in GitHub Packages if anonymous pulls should work.

For the full workflow reference — the publish pipeline, the local build scripts
it calls, the `gs-run` wrapper, and the verification contract — see
[`doc/workflows.md`](doc/workflows.md).

## PostScript integer width

The PostScript integer width column describes the behavior of the Docker images in this repository, not just the upstream Ghostscript release number. The local builds are not monotonic by version:

- Every pre-8.70 release in this repository (7.07, 8.01, 8.15, 8.50, 8.54, 8.60, 8.61, 8.62, 8.63, and 8.64) supports 64-bit PostScript integer objects. That era stored PostScript integer objects in a C `long`, which is 64-bit on x86_64/LP64, so values outside the signed 32-bit range stay `integertype`. This was confirmed by probing each image, not assumed from the release number.
- 8.71, 9.01, and 9.06 are 32-bit-only for PostScript integers in these images; values outside the 32-bit signed range are represented as `realtype`. Ghostscript made integer storage explicitly 32-bit starting at 8.70.
- 9.10 and later support 64-bit PostScript integer objects.

The pre-8.70 entries are distro package builds (Debian, Ubuntu, Fedora, and CentOS archives) except 8.63 and 8.64, which are source builds.

The repository does not currently include 9.07, 9.08, or 9.09.

## Versions

| Version  | Year | Source       | Base Image         | 64-bit PS integers | `gs` CPU arch |
|----------|------|--------------|--------------------|--------------------|---------------|
| 7.07     | 2003 | yum package  | centos:6           | Yes                | x86_64        |
| 8.01     | 2004 | apt package  | debian/eol:etch    | Yes                | x86_64        |
| 8.15     | 2005 | apt package  | debian/eol:etch    | Yes                | x86_64        |
| 8.50     | 2006 | apt package  | debian/eol:lenny   | Yes                | x86_64        |
| 8.54     | 2007 | apt package  | debian/eol:etch    | Yes                | x86_64        |
| 8.60     | 2007 | yum package  | centos:6           | Yes                | x86_64        |
| 8.61     | 2007 | apt package  | debian/eol:lenny   | Yes                | x86_64        |
| 8.62     | 2008 | yum package  | centos:6           | Yes                | x86_64        |
| 8.63     | 2008 | Source build | ubuntu:18.04       | Yes                | x86_64        |
| 8.64     | 2009 | Source build | ubuntu:18.04       | Yes                | x86_64        |
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

## Build notes

The pre-8.70 distro-package images (7.07–8.62) were constructed from archived
packages with some non-obvious dependency work (era base images, extracted old
sonames, font requirements). Each one is documented under
[`doc/build-notes/`](doc/build-notes/README.md): package source, base-image
choice, dependency closure, gotchas, and the verification probe.

## Adding a new version

1. Create a Dockerfile under `versions/<version>/Dockerfile`
2. Add the entry to the table above
3. Build: `docker build -t gs-<version> versions/<version>/`
4. Run: `docker run --rm -v "$(pwd):/work" gs-<version> <gs args>`
