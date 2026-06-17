# Debug and combined builds

How `gs-<v>:debug` and `gs-<v>:combined` are produced by `scripts/build-debug-image`
and `scripts/build-combined-image`. The policy: **prefer the distro's own debug
symbols; fall back to a source build only where no usable gs debug package
exists.** Every debug image (and, by inheritance, every combined image) also
ships the matching Ghostscript source under `/usr/src/ghostscript`.

The debug binary always ends up with a real `.debug_info` section in the `gs`
executable itself (the publish workflow checks this), so detached distro symbols
are recombined into the binary with `eu-unstrip`.

## Methods

| Method | Versions | How |
|--------|----------|-----|
| `deb` | 9.18, 9.22, 9.50 | Install `ghostscript-dbg` (Ubuntu xenial/bionic/focal main), resolve the gs/libgs build-ids, recombine with `eu-unstrip`. |
| `rpm-pm` | 9.26, 9.56.1, 10.03.1, 9.52 | Fetch the version-matched `ghostscript-debuginfo` from the Fedora/openSUSE archive (NEVR from `rpm -q`), recombine. |
| `debuginfo` | 7.07, 8.60, 8.62 | Fixed-URL `ghostscript-debuginfo` recombine — the normal image is an extracted RPM with no rpmdb, so the URL is hard-coded; source comes from the debuginfo's `/usr/src/debug`. |
| `source` | everything else | Compile from the upstream/orig tarball with `-g3 -O0`, building in `/usr/src/ghostscript`. |

For `deb`/`rpm-pm`/`source`, the source is the matching upstream/orig tarball
unpacked into `/usr/src/ghostscript`; for `debuginfo` it is the distro
`/usr/src/debug` tree symlinked to `/usr/src/ghostscript`.

## Why the source-build fallbacks

Distro-first was the goal for every version, but several distro debug packages
are unusable, so those fall back to source (still shipping source):

- **9.06 (Debian jessie):** `ghostscript-dbg` uses a path-style detached-symbol
  layout `eu-unstrip` rejects ("cannot find matching section").
- **9.10, 9.14, 9.20 (Fedora 20/21/25):** the archive cannot install `elfutils`
  (an `elfutils-libelf` dependency conflict), so the recombine can't run.
- **9.55.0, 10.02.1 (Ubuntu jammy/noble):** ddebs carries the `*-dbgsym` only for
  the GA revision, not the installed `-security` point release, so the build-ids
  don't match.
- **9.53.3, 10.0.0, 10.05.1 (Debian bullseye/bookworm/trixie) and 10.05.0
  (Ubuntu plucky):** no gs `dbgsym` is published at all.
- **8.63, 8.64, 8.71, 9.01, 10.03.0, 10.07.1:** the normal image is already a
  pure source build, so there is nothing distro to reuse.
- **8.01, 8.15, 8.50, 8.54, 8.61:** Debian/Ubuntu shipped no gs debug package in
  that era; built from the distro orig tarball (8.61 from the ghostpdl tag, whose
  orig fails to compile imdi on modern gcc).

## Recombine notes

- The gs/libgs detached symbols are located by build-id
  (`/usr/lib/debug/.build-id/XX/YYY.debug`), then by path-style
  (`/usr/lib/debug<path>.debug`), then by a `find` glob (openSUSE names its debug
  files with the package version).
- `file` must be installed for build-id extraction; the `rpm-pm` method installs
  it alongside `elfutils`.
- Old EOL bases (xenial, etc.) have a stale CA bundle, so the source tarball is
  fetched with `curl -k`.

## Combined-image fixes (`scripts/build-combined-image`)

Three changes were needed so `:combined` works for every version (all safe for
the others, and combined inherits `/usr/src/ghostscript` from the debug image):

1. `readlink -f` the gs path before copying (gs-gpl ships `/usr/bin/gs` as a
   symlink to `gs-gpl`).
2. Copy the `/usr/share/gs-gpl` resource tree (gs-gpl keeps `gs_init.ps` there).
3. Symlink `/opt/gs -> /opt/gs-debug` after the move so the source-built
   `gs-debug`'s compiled-in resource path resolves and it can render.

## Verification

For every version, both flavors are checked exactly as the publish workflow does:
correct `gs` / `gs-debug` `--version`, a `.debug_info` section in the relevant
executable, the `amd64`/`x86_64` and flavor labels, and a non-empty
`/usr/src/ghostscript`.
