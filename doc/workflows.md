# Workflows

The repository's automation in one place: the CI publish workflow, the local
build scripts it calls, and the run wrapper. Each version produces a **single
published image**, `gs-<version>:combined`, which contains the normal `gs`, the
debug `gs-debug`, `gdb`/`binutils`, and the Ghostscript source under
`/usr/src/ghostscript`. The normal and debug images are built only as
intermediate stages and are not published.

| Piece | File | Purpose |
|-------|------|---------|
| Publish workflow | `.github/workflows/publish-images.yml` | Build, verify, and push `:combined` to GHCR |
| Combined image | `scripts/build-combined-image` | Assemble `gs-<version>:combined` (the published image) |
| Debug stage | `scripts/build-debug-image` | Build the debug image used as the `gs-debug` source (intermediate) |
| Normal stage | `versions/<version>/Dockerfile` | Build the normal `gs` (intermediate) |
| Run wrapper | `gs-run` | Run the combined image locally, building on demand |

## CI: Publish Ghostscript Images

`.github/workflows/publish-images.yml`

- **Trigger:** manual only (`workflow_dispatch`). Input `version` is a single
  version (e.g. `9.50`) or `all`.
- **Jobs:**
  - `prepare` — turns the input into a matrix: one entry, or every directory
    under `versions/` (sorted) when `all`.
  - `publish` — runs per version, `max-parallel: 2`, `fail-fast: false`.
- **Per version, the `publish` job:**
  1. Builds `gs-<version>:combined` via `scripts/build-combined-image` (which
     builds the normal and debug stages it needs).
  2. Verifies `gs` and `gs-debug` `--version`, the Docker platform arch (`amd64`),
     the arch labels, `build-flavor=combined`, a `.debug_info` section in
     `gs-debug`, and the source under `/usr/src/ghostscript`.
  3. Pushes `:combined` to `ghcr.io/<owner>/gs-<version>` (retried up to 3×).
- **Auth:** the repository `GITHUB_TOKEN` with `packages: write`; no extra
  secrets. After the first publish, set the package visibility to public in
  GitHub Packages for anonymous pulls.
- **Quirk:** `10.0.0` reports its version as `10.00.0`; the workflow expects that.

## Local: the combined image

```bash
scripts/build-combined-image <version> [image-tag]  # default tag gs-<version>:combined
```

Assembles the published image from two intermediate builds (kept local):

1. **Normal** — `docker build versions/<version>/` → the `gs` binary. Each
   Dockerfile is a distro-package build (apt / yum / dnf / zypper, sometimes
   extracting archived packages) or a source build. Construction details for the
   pre-8.70 versions are in [`build-notes/`](build-notes/README.md).
2. **Debug** — `scripts/build-debug-image <version>` → the `gs-debug` binary (see
   below).

The combined image installs the normal `gs` (via a wrapper at `/usr/local/bin/gs`)
and `gs-debug` at `/usr/local/bin/gs-debug`, inherits the debug image's
`/usr/src/ghostscript`, and carries `gdb`/`binutils`. Defaults to two compiler
jobs (`GS_BUILD_JOBS`).

## The debug stage

```bash
scripts/build-debug-image <version> [image-tag]   # default tag gs-<version>:debug
```

Produces an unstripped `gs` with a `.debug_info` section plus the matching source
under `/usr/src/ghostscript`. It prefers distro debug symbols and falls back to
source only when no usable gs debug package exists:

- **`deb`** (Debian/Ubuntu, distro symbols) — install `ghostscript-dbg` /
  `*-dbgsym` and recombine into `gs`/`libgs` with `eu-unstrip`. Used for 9.18,
  9.22, 9.50.
- **`rpm-pm`** (Fedora/openSUSE, distro symbols) — fetch the version-matched
  `ghostscript-debuginfo` from the archive and recombine. Used for 9.26, 9.56.1,
  10.03.1, 9.52.
- **`debuginfo`** (CentOS-hosted RPM payloads) — fixed-URL `ghostscript-debuginfo`
  recombine, because the normal image is an extracted RPM with no rpmdb. Used for
  7.07, 8.60, 8.62.
- **`source`** (fallback + pure-source) — compile from the upstream/orig tarball
  with `-g3 -O0 -fno-omit-frame-pointer`. Used for 8.63/8.64/8.71/9.01/10.03.0/
  10.07.1, the pre-8.63 deb versions (8.01/8.15/8.50/8.54/8.61), and the distro
  versions with no usable debug package (9.06, 9.10, 9.14, 9.20, 9.53.3, 9.55.0,
  10.0.0, 10.02.1, 10.05.0, 10.05.1).

See [`build-notes/debug-and-combined.md`](build-notes/debug-and-combined.md) for
the per-version specifics.

## Run wrapper

```bash
./gs-run                                 # list available versions
./gs-run <version> -- <gs args>          # run gs in gs-<version>:combined
./gs-run --debug <version> -- <gs args>  # run gs-debug in gs-<version>:combined
```

The wrapper builds `gs-<version>:combined` on first use (via
`scripts/build-combined-image`), mounts the current directory at `/work`, and
forwards `<gs args>` to the entrypoint.

## Verification contract

The published `gs-<version>:combined` image must satisfy, in both CI and local
checks:

- `gs --version` and `gs-debug --version` equal the version.
- `org.opencontainers.image.architecture=amd64`,
  `net.w0ot.ghostscript.cpu-architecture=x86_64`, and
  `net.w0ot.ghostscript.build-flavor=combined` labels are present.
- The `gs-debug` executable has a `.debug_info` section.
- The image carries the Ghostscript source under `/usr/src/ghostscript`.

## Adding a version

1. Add `versions/<version>/Dockerfile` (the normal `gs` build).
2. If it is not covered by an existing range, add a case to
   `scripts/build-debug-image`.
3. Build and verify the combined image locally
   (`scripts/build-combined-image <version>`), or run the publish workflow for
   that single version.
4. Update the `README.md` versions table; add a `doc/build-notes/` page if the
   build is non-obvious.
