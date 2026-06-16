# TODO: pre-2008 distro package candidates

This is a research list only. Do not treat these as implemented versions until
each candidate has a Dockerfile, a successful image build, and a `gs --version`
verification.

The goal is to fill the 2001-2007 gap with distro package builds where practical,
instead of defaulting to source builds. Older distro packages may be i386-only;
those should be tagged with the hosted `gs` CPU architecture if implemented.

## Candidate packages

| Target year | Candidate `gs` version | Distro source | Package | Arch | Notes |
|-------------|------------------------|---------------|---------|------|-------|
| 2001 | 6.51 | CentOS 2.1 final vault | `ghostscript-6.51-16.3.i386.rpm` | i386 | Direct binary RPM is available. Likely needs an i386 runtime and old RPM-era dependencies such as `ghostscript-fonts`, `Omni`, and old glibc libraries. |
| 2002 | 6.53 | Debian 3.0 Woody archive | `gs_6.53-3_i386.deb` | i386 | Direct Debian archive metadata exists. Woody has no official amd64 package, so this would be i386 unless rebuilt from source. |
| 2003 | 7.05 | CentOS/RHEL 3 updates | `ghostscript-7.05-32.1.10.i386.rpm` | i386 | Direct CentOS vault binary RPM is available. Use this as the first 7.05 package candidate. |
| 2004 | 8.01 | Ubuntu 4.10 Warty old-releases | `gs-gpl_8.01-4_amd64.deb` | x86_64 | Direct amd64 deb is available. This is likely easier than using i386 Debian Sarge for 8.01. |
| 2005 | 8.15 | Ubuntu 6.06 Dapper old-releases | `gs-gpl_8.15-4ubuntu3_amd64.deb` | x86_64 | Direct amd64 deb is available. Fedora/EL packages also have 8.15 variants, but Ubuntu looks simpler. |
| 2006 | 8.50 | Ubuntu 6.10 Edgy old-releases | `gs-gpl_8.50-1.1ubuntu1_amd64.deb` | x86_64 | Direct amd64 deb is available. |
| 2007 | 8.54 | Ubuntu 7.04 Feisty old-releases or Debian 4.0 Etch archive | `gs-gpl_8.54.dfsg.1-5build1_amd64.deb` or `gs-gpl_8.54.dfsg.1-5etch2_amd64.deb` | x86_64 | Direct amd64 debs are available from both Ubuntu and Debian. |
| 2007 | 8.61 | Ubuntu 7.10 Gutsy old-releases | `ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb` | x86_64 | Direct amd64 deb is available. This may be a useful late-2007 checkpoint before the existing 8.63 image. |

## Follow-up implementation tasks

- Decide whether the pre-amd64 candidates should be implemented as i386 images or replaced with source-built x86_64 images.
- If implementing i386 images, update the architecture label strategy so `net.w0ot.ghostscript.cpu-architecture` can be `i386` or `i686` instead of always `x86_64`.
- Prototype one RPM-era image first, probably `6.51` or `7.05`, and verify whether the old RPM dependency graph can be satisfied from a vault base image.
- Prefer old distro package repositories over downloading a single package file when that keeps dependencies reproducible.
- For each implemented candidate, add `versions/<version>/Dockerfile`, update the README versions table, build the image, and verify `gs --version`.
- After each version is added, probe PostScript integer width and document the result rather than assuming behavior from the upstream release number.

## References checked

- CentOS 2.1 final vault has `ghostscript-6.51-16.3.i386.rpm`:
  `https://ftp.iij.ad.jp/pub/linux/centos-vault/2.1/final/i386/CentOS/RPMS/ghostscript-6.51-16.3.i386.rpm`
- Debian 3.0 Woody archive metadata lists `gs_6.53-3_i386.deb`:
  `http://archive.debian.org/debian/dists/woody/main/binary-i386/Packages.gz`
- CentOS 3.5 updates vault has `ghostscript-7.05-32.1.10.i386.rpm`:
  `https://ftp.iij.ad.jp/pub/linux/centos-vault/3.5/updates/i386/RPMS/ghostscript-7.05-32.1.10.i386.rpm`
- Scientific Linux/CentOS 4-era vaults have `ghostscript-7.07-33.2.el4_6.1.x86_64.rpm`:
  `https://ftp.iij.ad.jp/pub/linux/centos-vault/4.6/updates/x86_64/RPMS/ghostscript-7.07-33.2.el4_6.1.x86_64.rpm`
- Ubuntu old-releases metadata lists the amd64 `gs-gpl` and `ghostscript` candidates for Warty through Gutsy:
  `http://old-releases.ubuntu.com/ubuntu/dists/<suite>/main/binary-amd64/Packages.gz`
