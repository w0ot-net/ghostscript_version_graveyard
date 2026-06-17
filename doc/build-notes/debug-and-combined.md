# Debug and combined builds for the pre-8.70 versions

How `gs-<v>:debug` and `gs-<v>:combined` were made to work for 7.07–8.62, which
do not fit the standard `scripts/build-debug-image` source path used by the
newer versions. Two methods are used, split by package type.

## deb-based versions: source build (8.01, 8.15, 8.50, 8.54, 8.61)

Debian/Ubuntu shipped **no** `gs` debug package in this era (no `gs-gpl-dbg` /
`ghostscript-dbg`; the binaries are stripped), so the debug companion is
compiled from source with `-g3 -O0 -fno-omit-frame-pointer`, on `ubuntu:18.04`
like the other source builds.

Gotchas found:

- **Version mismatch from GitHub tags.** The `ArtifexSoftware/ghostpdl` tags
  `ghostscript-8.01` and `ghostscript-8.15` build binaries that report `8.00`
  and `8.01` (Artifex lineage ≠ the GNU/ESP Ghostscript the distro shipped). The
  debug companion's `gs --version` must match the normal image, so 8.01/8.15/8.50
  are built from the distro **orig tarball**
  (`gs-gpl_<v>.orig.tar.gz`) instead, which reports the correct version.
- **8.54 has no GitHub tag** at all → built from the Debian
  `gs-gpl_8.54.dfsg.1.orig.tar.gz`.
- **8.61 orig fails to compile** the imdi interpolation tables on modern gcc, but
  the ghostpdl GitHub tag `ghostscript-8.61` builds cleanly and reports 8.61, so
  8.61 uses the GitHub tag (autogen) rather than the orig tarball.
- **Missing bundled third-party sources.** These old trees expect bundled
  `jpeg`/`libpng`/`zlib` sources that the exports omit, so `configure` fails with
  "I wasn't able to find a copy of the jpeg library". Fixed by installing
  `libjpeg-dev libpng-dev zlib1g-dev libtiff-dev` (the `extra_apt` field).
- Orig tarballs unpack to varying top directories (`gs-gpl-8.01` vs
  `gs-gpl-8.15.orig`), so the script auto-detects the top dir (`source_dir=AUTO`).

## RPM-based versions: debuginfo recombine (7.07, 8.60, 8.62)

These have a real `ghostscript-debuginfo` package, so there is no need to
recompile. The debug image is built `FROM gs-<v>` (the normal image) and the
stripped `gs` and `libgs.so` are recombined with their detached debug symbols
using `eu-unstrip` (elfutils):

```
eu-unstrip /usr/bin/gs            <gs.debug>    -o /opt/gs/bin/gs
eu-unstrip /usr/lib64/libgs.so.X  <libgs.debug> -o (replace in place)
```

This yields a runnable, unstripped `gs` whose own binary carries a real
`.debug_info` section (what the publish workflow checks) and that matches the
exact shipped binary. Debuginfo sources:

- 7.07 → `http://debuginfo.centos.org/4/x86_64/ghostscript-debuginfo-7.07-...rpm`
- 8.60 → Fedora 8 archive `…/8/Everything/x86_64/debug/ghostscript-debuginfo-…`
- 8.62 → Fedora 9 archive `…/9/Everything/x86_64/debug/ghostscript-debuginfo-…`

## Combined-image fixes (`scripts/build-combined-image`)

Making `:combined` work for these versions needed three changes to the shared
combined script (all safe for the existing versions):

1. **Resolve the gs symlink.** gs-gpl ships `/usr/bin/gs` as a symlink to
   `gs-gpl` (via `gs-common`'s alternatives for 8.54). The script now does
   `readlink -f` before copying the binary, so it copies the real ELF instead of
   a dangling link.
2. **Copy the `/usr/share/gs-gpl` resource tree.** gs-gpl keeps `gs_init.ps`
   under `/usr/share/gs-gpl/<v>/lib`, not `/usr/share/ghostscript`, so that path
   was added to the resource copy/symlink lists.
3. **Keep `/opt/gs` pointing at the moved debug tree.** After
   `mv /opt/gs /opt/gs-debug`, a symlink `/opt/gs -> /opt/gs-debug` is added so
   the source-built `gs-debug`'s compiled-in search path still resolves and
   `gs-debug` can actually render (previously it only answered `--version`). This
   also fixes `gs-debug` rendering for the existing source-built versions.

## Verification

For every version, both the debug and combined images were checked exactly as
the publish workflow does:

- `gs --version` and `gs-debug --version` equal the expected version
- `readelf -S` shows a `.debug_info` section in the `gs` (debug) and `gs-debug`
  (combined) executables
- `amd64` / `x86_64` arch labels and the `debug` / `combined` flavor labels
- plus the 64-bit integer probe runs through both `gs` and `gs-debug`.
