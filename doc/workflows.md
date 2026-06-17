# Workflows

The repository's automation in one place: the CI publish workflow, the local
build scripts it calls, and the run wrapper. Each version produces three image
flavors — `latest` (release), `debug`, and `combined`.

| Piece | File | Purpose |
|-------|------|---------|
| Publish workflow | `.github/workflows/publish-images.yml` | Build, verify, and push all three flavors to GHCR |
| Release image | `versions/<version>/Dockerfile` | The normal `gs-<version>:latest` image |
| Debug image | `scripts/build-debug-image` | Build the `gs-<version>:debug` companion |
| Combined image | `scripts/build-combined-image` | Build `gs-<version>:combined` (`gs` + `gs-debug`) |
| Run wrapper | `gs-run` | Run any flavor locally, building on demand |

## CI: Publish Ghostscript Images

`.github/workflows/publish-images.yml`

- **Trigger:** manual only (`workflow_dispatch`). Input `version` is a single
  version (e.g. `9.50`) or `all`.
- **Jobs:**
  - `prepare` — turns the input into a matrix: one entry, or every directory
    under `versions/` (sorted) when `all`.
  - `publish` — runs per version, `max-parallel: 2`, `fail-fast: false`.
- **Per version, the `publish` job:**
  1. Builds `versions/<version>/Dockerfile` → `gs-<version>` and verifies
     `gs --version`, the Docker platform arch (`amd64`), and the arch labels.
  2. Builds the debug image via `scripts/build-debug-image` and verifies version,
     arch/labels, `build-flavor=debug`, and a `.debug_info` section in `gs`.
  3. Builds the combined image via `scripts/build-combined-image` and verifies
     `gs` and `gs-debug` versions, arch/labels, `build-flavor=combined`, and a
     `.debug_info` section in `gs-debug`.
  4. Pushes `:latest`, `:debug`, and `:combined` to
     `ghcr.io/<owner>/gs-<version>` (each push retried up to 3×).
- **Auth:** the repository `GITHUB_TOKEN` with `packages: write`; no extra
  secrets. After the first publish, set the package visibility to public in
  GitHub Packages for anonymous pulls.
- **Quirk:** `10.0.0` reports its version as `10.00.0`; the workflow expects that.

## Local: release image

```bash
docker build -t gs-<version> versions/<version>/
```

Each `versions/<version>/Dockerfile` is either a distro-package build (apt / yum /
dnf / zypper, sometimes extracting archived packages) or a source build, and
carries the `org.opencontainers.image.architecture=amd64` and
`net.w0ot.ghostscript.cpu-architecture=x86_64` labels. Construction details for
the pre-8.70 distro-package versions are in [`build-notes/`](build-notes/README.md).

## Local: debug image

```bash
scripts/build-debug-image <version> [image-tag]   # default tag gs-<version>:debug
```

Produces an unstripped `gs` with a `.debug_info` section plus `gdb`/`binutils`,
built `-g3 -O0 -fno-omit-frame-pointer`. Two methods, chosen per version:

- **Source build** (most versions) — compile from an upstream tarball: a GitHub
  release tag for the newer versions and 8.61, or the distro orig tarball for the
  pre-8.63 deb versions (8.01, 8.15, 8.50, 8.54).
- **Debuginfo recombine** (7.07, 8.60, 8.62) — recombine the stripped distro
  binary with its `ghostscript-debuginfo` package via `eu-unstrip`.

See [`build-notes/debug-and-combined.md`](build-notes/debug-and-combined.md) for
the per-version specifics. Set `GS_BUILD_JOBS` to control compile parallelism
(default 2; legacy autoconf builds force 1).

## Local: combined image

```bash
scripts/build-combined-image <version> [image-tag] # default tag gs-<version>:combined
```

Assembles one image from the normal and debug companions: `gs` is the normal
(distro/source) binary, `gs-debug` is the debug binary at `/usr/local/bin/gs-debug`.
It builds the normal and debug images first if they are missing. Defaults to 2
compiler jobs (`GS_BUILD_JOBS`).

## Run wrapper

```bash
./gs-run                                   # list available versions
./gs-run <version> -- <gs args>            # normal (gs-<version>)
./gs-run --debug <version> -- <gs args>    # gs-<version>:debug
./gs-run --combined <version> -- <gs args> # gs-<version>:combined, runs gs
./gs-run --combined-debug <version> -- ... # gs-<version>:combined, runs gs-debug
```

The wrapper builds the image on first use (via the matching script), mounts the
current directory at `/work`, and forwards `<gs args>` to the entrypoint.

## Verification contract

Every flavor of every version must satisfy, in both CI and local checks:

- `gs --version` (and `gs-debug --version` for combined) equals the version.
- `org.opencontainers.image.architecture=amd64` and
  `net.w0ot.ghostscript.cpu-architecture=x86_64` labels are present; the debug
  and combined images also carry the `build-flavor` label.
- The `gs` executable (debug) / `gs-debug` executable (combined) has a
  `.debug_info` section.

## Adding a version

1. Add `versions/<version>/Dockerfile`.
2. If it is not covered by the existing ranges, add a case to
   `scripts/build-debug-image`.
3. Build and verify all three flavors locally (or run the publish workflow for
   that single version).
4. Update the `README.md` versions table; add a `doc/build-notes/` page if the
   build is non-obvious.
