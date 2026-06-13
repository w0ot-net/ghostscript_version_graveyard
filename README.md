# ghostscript_version_graveyard

A collection of Ghostscript binaries spanning different versions and years, each packaged in its own Docker container so they can all run on a single host without conflicts.

## Why

Ghostscript behavior varies across versions — rendering quirks, security patches, PostScript/PDF interpreter changes. Having a graveyard of old versions lets you:

- Reproduce bugs tied to a specific Ghostscript release
- Compare output across versions (regression testing, fuzz testing)
- Test how a document renders on the same Ghostscript version a particular distro ships

## How it works

Each Ghostscript version gets its own Docker image built from a base distro that originally shipped it (e.g. Ubuntu 14.04 for GS 9.10, Debian Stretch for GS 9.20). This ensures the binary runs against the same libc and shared libraries it was built for.

A wrapper script lets you invoke any version by name:

```
./gs-run 9.26 -- -dBATCH -dNOPAUSE -sDEVICE=pdfwrite -sOutputFile=out.pdf input.ps
```

## Versions

<!-- Populated as versions are added -->

| Version | Source        | Base Image       |
|---------|---------------|------------------|
| ...     | ...           | ...              |

## Adding a new version

1. Create a Dockerfile under `versions/<version>/Dockerfile`
2. Add the entry to the table above
3. Build: `docker build -t gs-<version> versions/<version>/`
4. Run: `docker run --rm -v $(pwd):/work gs-<version> gs <args>`
