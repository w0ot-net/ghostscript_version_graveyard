# ghostscript_version_graveyard

A collection of Ghostscript binaries spanning different versions and years, each packaged in its own Docker container so they can all run on a single host without conflicts.

## Why

Ghostscript behavior varies across versions — rendering quirks, security patches, PostScript/PDF interpreter changes. Having a graveyard of old versions lets you:

- Reproduce bugs tied to a specific Ghostscript release
- Compare output across versions (regression testing, fuzz testing)
- Test how a document renders on the same Ghostscript version a particular distro ships

## How it works

Each Ghostscript version gets its own Docker image. Older versions are compiled from source on a base image with a compatible libc; newer versions use distro packages directly.

A wrapper script lets you invoke any version by name (auto-builds the image on first run):

```
./gs-run 9.50 -- -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=out.pdf input.ps
```

## Versions

| Version  | Source       | Base Image         |
|----------|--------------|---------------------|
| 8.63     | Source build | ubuntu:18.04        |
| 8.64     | Source build | ubuntu:18.04        |
| 9.01     | Source build | ubuntu:18.04        |
| 9.14     | Source build | ubuntu:22.04        |
| 9.18     | Source build | ubuntu:22.04        |
| 9.50     | apt package  | ubuntu:20.04        |
| 9.52     | zypper pkg   | opensuse/leap:15.5  |
| 9.53.3   | apt package  | debian:bullseye     |
| 9.55.0   | apt package  | ubuntu:22.04        |
| 9.56.1   | dnf package  | fedora:36           |
| 10.0.0   | apt package  | debian:bookworm     |
| 10.02.1  | apt package  | ubuntu:24.04        |
| 10.03.0  | Source build | ubuntu:24.04        |
| 10.03.1  | dnf package  | fedora:41           |
| 10.05.0  | apt package  | ubuntu:25.04        |
| 10.05.1  | apt package  | debian:trixie       |
| 10.07.1  | Source build | ubuntu:24.04        |

## Adding a new version

1. Create a Dockerfile under `versions/<version>/Dockerfile`
2. Add the entry to the table above
3. Build: `docker build -t gs-<version> versions/<version>/`
4. Run: `docker run --rm -v $(pwd):/work gs-<version> gs <args>`
