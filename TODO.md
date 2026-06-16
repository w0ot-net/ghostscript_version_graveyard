# TODO: x64 distro builds expected to support 64-bit PS integers

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

| Version | Distro package source | Package | Arch | Why it qualifies |
|---------|-----------------------|---------|------|------------------|
| 7.07 | CentOS 4.6 vault updates | `ghostscript-7.07-33.2.el4_6.1.x86_64.rpm` | x86_64 | x86_64 distro RPM; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.01 | Ubuntu 4.10 Warty old-releases | `gs-gpl_8.01-4_amd64.deb` | x86_64 | amd64 distro deb; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.15 | Ubuntu 6.06 Dapper old-releases | `gs-gpl_8.15-4ubuntu3_amd64.deb` | x86_64 | amd64 distro deb; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.50 | Ubuntu 6.10 Edgy old-releases | `gs-gpl_8.50-1.1ubuntu1_amd64.deb` | x86_64 | amd64 distro deb; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.54 | Debian 4.0 Etch archive | `gs-gpl_8.54.dfsg.1-5etch2_amd64.deb` | x86_64 | amd64 distro deb; source confirms `ref.value.intval` is C `long` in 8.54. |
| 8.54 | Ubuntu 7.04 Feisty old-releases | `gs-gpl_8.54.dfsg.1-5build1_amd64.deb` | x86_64 | Alternate amd64 distro deb for 8.54. Prefer either Debian Etch or Ubuntu Feisty, not both, unless both are useful for comparison. |
| 8.60 | Fedora 8 archive | `ghostscript-8.60-5.fc8.x86_64.rpm` | x86_64 | x86_64 distro RPM; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.61 | Ubuntu 7.10 Gutsy old-releases | `ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb` | x86_64 | amd64 distro deb; pre-8.70 integer storage should use 64-bit `long` on LP64. |
| 8.62 | Fedora 9 archive | `ghostscript-8.62-3.fc9.x86_64.rpm` | x86_64 | x86_64 distro RPM; last obvious distro-package candidate before the existing 8.63 source build. |

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
