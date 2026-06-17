# TODO: x64 distro builds expected to support 64-bit PS integers

**Status: DONE (2026-06-17).** All candidates below were implemented, built, and
probed. Every one keeps a value outside the signed 32-bit range as `integertype`,
confirming 64-bit PostScript integer storage. The Ubuntu 7.04 Feisty alternate
for 8.54 was intentionally skipped in favor of the Debian 4.0 Etch build. See the
`README.md` versions table and integer-width notes for the results, and
`versions/<version>/Dockerfile` for each build.

Add Ghostscript versions that satisfy all of these constraints:

- distro package build, not a local source build
- x64 hosted `gs` binary
- supports 64-bit PostScript integer objects

This list is based on package artifacts that were found in distro archives.
The 64-bit integer expectation comes from Ghostscript's pre-8.70 behavior:
older releases stored PostScript integer objects in C `long`, which is 64-bit
on Linux x86_64/LP64. Ghostscript 8.70 later made this explicitly 32-bit, so
do not assume 8.70 or later satisfies this TODO unless it is separately proved.

Every candidate below must still be probed in Docker before being marked
complete in `README.md`.

## Candidates

| Version | Distro package source | Package | Arch | Status | Build base |
|---------|-----------------------|---------|------|--------|------------|
| 7.07 | CentOS 4.6 vault updates | `ghostscript-7.07-33.2.el4_6.1.x86_64.rpm` | x86_64 | Done, 64-bit confirmed | `centos:6` (RPM extracted; VFlib2 + el4 libttf from vault) |
| 8.01 | Ubuntu 4.10 Warty old-releases | `gs-gpl_8.01-4_amd64.deb` | x86_64 | Done, 64-bit confirmed | `debian/eol:etch` (deb extracted) |
| 8.15 | Ubuntu 6.06 Dapper old-releases | `gs-gpl_8.15-4ubuntu3_amd64.deb` | x86_64 | Done, 64-bit confirmed | `debian/eol:etch` (deb extracted) |
| 8.50 | Ubuntu 6.10 Edgy old-releases | `gs-gpl_8.50-1.1ubuntu1_amd64.deb` | x86_64 | Done, 64-bit confirmed | `debian/eol:lenny` (needs glibc 2.4+; deb extracted) |
| 8.54 | Debian 4.0 Etch archive | `gs-gpl_8.54.dfsg.1-5etch2_amd64.deb` | x86_64 | Done, 64-bit confirmed | `debian/eol:etch` (repo-backed apt) |
| 8.54 | Ubuntu 7.04 Feisty old-releases | `gs-gpl_8.54.dfsg.1-5build1_amd64.deb` | x86_64 | Skipped (used Debian Etch instead) | — |
| 8.60 | Fedora 8 archive | `ghostscript-8.60-5.fc8.x86_64.rpm` | x86_64 | Done, 64-bit confirmed | `centos:6` (RPM extracted) |
| 8.61 | Ubuntu 7.10 Gutsy old-releases | `ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb` | x86_64 | Done, 64-bit confirmed | `debian/eol:lenny` (libgs8 closure; libgnutls13 from gutsy) |
| 8.62 | Fedora 9 archive | `ghostscript-8.62-3.fc9.x86_64.rpm` | x86_64 | Done, 64-bit confirmed | `centos:6` (RPM extracted) |

## Implementation checklist

- Prefer repo-backed installs from the archived distro where dependency
  resolution is practical.
- If repo-backed install is not practical, download exact public package files
  and document the dependency closure in the Dockerfile.
- Tag the hosted `gs` CPU architecture as `x86_64`.
- Build each image as `gs-<version>:latest`.
- Verify `gs --version` returns the expected version.
- Probe integer width in the built container. The candidate is only done if a
  value outside signed 32-bit range remains `integertype`, not `realtype`.
- Update the `README.md` versions table and integer-width notes after each
  successful version.
- Commit and push each implemented version separately.

## Package URLs checked

- `https://ftp.iij.ad.jp/pub/linux/centos-vault/4.6/updates/x86_64/RPMS/ghostscript-7.07-33.2.el4_6.1.x86_64.rpm`
- `http://old-releases.ubuntu.com/ubuntu/pool/main/g/gs-gpl/gs-gpl_8.01-4_amd64.deb`
- `http://old-releases.ubuntu.com/ubuntu/pool/main/g/gs-gpl/gs-gpl_8.15-4ubuntu3_amd64.deb`
- `http://old-releases.ubuntu.com/ubuntu/pool/main/g/gs-gpl/gs-gpl_8.50-1.1ubuntu1_amd64.deb`
- `http://archive.debian.org/debian/pool/main/g/gs-gpl/gs-gpl_8.54.dfsg.1-5etch2_amd64.deb`
- `http://old-releases.ubuntu.com/ubuntu/pool/main/g/gs-gpl/gs-gpl_8.54.dfsg.1-5build1_amd64.deb`
- `https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/8/Everything/x86_64/os/Packages/ghostscript-8.60-5.fc8.x86_64.rpm`
- `http://old-releases.ubuntu.com/ubuntu/pool/main/g/ghostscript/ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb`
- `https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/9/Everything/x86_64/os/Packages/ghostscript-8.62-3.fc9.x86_64.rpm`

# TODO: ship Ghostscript source code inside every debug/combined image

We want the corresponding Ghostscript source tree present inside **every**
`gs-<version>:debug` and `gs-<version>:combined` image, so the debug symbols can
be matched to source while debugging.

Current state (not done — inconsistent and partly accidental):

- Source builds 8.01, 8.15, 8.50, 8.54 happen to leave the source in
  `/tmp/gs-gpl-<v>/` only because the cleanup line in `scripts/build-debug-image`
  does not match that directory name.
- All other source builds (8.61 and the existing 8.63–10.07.1 versions) delete
  the source at the end of the build (`rm -rf /tmp/...`).
- The debuginfo-based versions (7.07, 8.60, 8.62) keep no source: the
  `ghostscript-debuginfo` RPM's `/usr/src/debug` tree is extracted to a temp dir
  and then removed; only the `.debug` symbols are kept.

What to do:

- For source builds: keep the unpacked source tree at a stable, documented path
  (e.g. `/usr/src/ghostscript`) instead of deleting it, for every version.
- For debuginfo builds (7.07, 8.60, 8.62): install the `/usr/src/debug` source
  that ships in `ghostscript-debuginfo` (and ideally relocate/symlink it to the
  same stable path) so gdb can find it.
- Make it consistent across `:debug` and `:combined` (combined inherits from the
  debug image, so fixing debug covers combined).
- Consider a publish-workflow check that the source path exists in both flavors.
- Document the source path in `README.md` and `doc/build-notes/`.
