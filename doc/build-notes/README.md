# Build notes: pre-8.70 x64 distro packages (7.07–8.62)

These notes record how the eight pre-8.70 distro-package images were constructed,
why each base image was chosen, the exact dependency closure, the gotchas hit
during construction, and how each was verified. They cover the versions added for
the "x64 distro builds expected to support 64-bit PS integers" TODO.

One file per version:

| Version | Package source | Base image | Method | Notes |
|---------|----------------|------------|--------|-------|
| [7.07](7.07.md) | CentOS 4.6 vault updates RPM | `centos:6` | RPM extracted | extinct VFlib2 + FreeType1 `libttf.so.2`; needs URW fonts |
| [8.01](8.01.md) | Ubuntu 4.10 warty deb | `debian/eol:etch` | deb extracted | needs `libgimpprint1` from warty |
| [8.15](8.15.md) | Ubuntu 6.06 dapper deb | `debian/eol:etch` | deb extracted | png/jpeg sonames from etch |
| [8.50](8.50.md) | Ubuntu 6.10 edgy deb | `debian/eol:lenny` | deb extracted | needs glibc ≥ 2.4 (etch too old) |
| [8.54](8.54.md) | Debian 4.0 etch deb | `debian/eol:etch` | repo-backed apt | cleanest; native etch package |
| [8.60](8.60.md) | Fedora 8 archive RPM | `centos:6` | RPM extracted | — |
| [8.61](8.61.md) | Ubuntu 7.10 gutsy deb | `debian/eol:lenny` | deb extracted | `libgs8` closure; `libgnutls13` from gutsy |
| [8.62](8.62.md) | Fedora 9 archive RPM | `centos:6` | RPM extracted | + `libjasper.so.1` |

## The 64-bit integer premise

The TODO's reason for collecting these versions: Ghostscript before 8.70 stored
PostScript integer objects in a C `long`, which is 64-bit on x86_64/LP64. 8.70
later made integer storage explicitly 32-bit. So every pre-8.70 release built on
an LP64 host is expected to keep values outside the signed 32-bit range as
`integertype` rather than promoting them to `realtype`.

This is an *expectation*, not an assumption: every image is probed.

## Verification probe

Each image is verified two ways:

```bash
# 1. version string
docker run --rm --entrypoint gs gs-<v> --version        # must equal <v>

# 2. integer width probe (64-bit => integertype, 32-bit => realtype)
docker run --rm --entrypoint gs gs-<v> \
  -dBATCH -dNOPAUSE -dNODISPLAY -q -c "5000000000 dup type == == quit"
# 64-bit result:
#   integertype
#   5000000000
```

`5000000000` is `0x12A05F200`, above the signed 32-bit maximum (`2147483647`).
On a 32-bit-integer build the scanner stores it as `realtype` and prints
`5e+09`. The probe was first validated against the existing reference images
(8.63 → `integertype`, 8.71 → `realtype`) before trusting it on new builds.

`-dNODISPLAY` is required: these old builds default to the X11 device and abort
with "Cannot open X display" otherwise.

## Why "era base" images

None of warty/dapper/edgy/etch/gutsy or Fedora 8/9 or CentOS 4 has a usable
Docker base image. Rather than fight a modern glibc and missing old shared
libraries, each binary is hosted on the *nearest archive base whose glibc is new
enough to run it and that still ships the old sonames it needs*:

- `debian/eol:etch` (glibc 2.3.6) — hosts the warty/dapper gs-gpl binaries and
  the native etch package. Still ships `libpng12`, `libjpeg62`, `libgimpprint1`,
  `libpaper1`, and the old X sonames.
- `debian/eol:lenny` (glibc 2.7) — used when the binary needs `libc6 >= 2.4`
  (edgy 8.50) or the larger gutsy `libgs8` closure (8.61).
- `centos:6` (glibc 2.12) — hosts the Fedora 8/9 and CentOS 4 RPM payloads.
  Still ships `libtiff.so.3`, `libjpeg.so.62`, `libpng12.so.0`, `libjasper.so.1`,
  the krb5 sonames, and `libcups*`. The `.repo` files are pointed at
  `vault.centos.org` because CentOS 6 is EOL.

## Shared construction pattern

1. Identify the gs binary's runtime needs with `readelf -d <gs> | grep NEEDED`
   (and the same on `libgs.so.*` when the package splits out a library).
2. Satisfy as many sonames as possible from the base distro's own package
   manager (apt/yum against the archive/vault).
3. For sonames the base lacks, download the exact period package providing them
   and extract it (`dpkg-deb -x` / `rpm2cpio | cpio`) onto the image.
4. Extract the gs package itself, `ldconfig`, ensure `/usr/bin/gs` resolves
   (symlink to `gs-gpl` for the gs-gpl debs).
5. `ldd /usr/bin/gs | grep 'not found'` must be empty before probing.

`binutils` and `gdb` are installed in every image, matching the other
package-based images in this repository.

## Per-version gotchas (summary)

- **7.07** needs two extinct font-engine libraries — `libVFlib2.so.24` (VFlib2)
  and `libttf.so.2` (FreeType 1, which the el4 `freetype` package bundled
  alongside `libfreetype.so.6`) — pulled from the CentOS 4 vault. It also aborts
  at startup unless `urw-fonts` and `ghostscript-fonts` are installed.
- **8.01** links `libgimpprint.so.1`; that package is not in etch, so the warty
  `libgimpprint1` deb is extracted.
- **8.50** requires `libc6 >= 2.4`; etch's glibc 2.3.6 fails with
  `GLIBC_2.4 not found`, so it is hosted on lenny.
- **8.61** splits into a `ghostscript` front-end + `libgs8`; `libgs.so.8` pulls a
  large closure. Lenny supplies all of it except `libgnutls.so.13`, which is
  extracted from the gutsy archive.
