# Ghostscript 7.05 from the CentOS 3.9 vault updates (ghostscript x86_64 RPM).
# CentOS 3 has no usable base image, so the gs binary is extracted onto a CentOS 6
# base (glibc 2.12). CentOS 6 supplies most runtime sonames, but several extinct
# 2003-era libraries are pulled from distro vaults: libttf.so.2 (FreeType 1) and
# libVFlib2.so.24 from the CentOS 4 vault (as for 7.07), and glib 1.2
# (libglib-1.2/libgmodule-1.2) plus libgimpprint.so.1 from the CentOS 3.9 vault.
# libstdc++.so.5 comes from compat-libstdc++-33. urw-fonts + ghostscript-fonts are
# required or gs aborts at startup.
FROM centos:6

ARG VAULT4=https://ftp.iij.ad.jp/pub/linux/centos-vault/4.6/os/x86_64/CentOS/RPMS
ARG VAULT3=https://vault.centos.org/3.9/os/x86_64/RedHat/RPMS
ARG GS=https://vault.centos.org/3.9/updates/x86_64/RPMS/ghostscript-7.05-32.1.20.x86_64.rpm

RUN sed -i 's|^mirrorlist=|#mirrorlist=|; s|^#baseurl=http://mirror.centos.org|baseurl=https://vault.centos.org|' /etc/yum.repos.d/CentOS-Base.repo && \
    yum -y install binutils gdb libpng libjpeg-turbo cups-libs compat-libstdc++-33 zlib \
        libX11 libXext libXt libSM libICE urw-fonts ghostscript-fonts && \
    yum clean all && \
    cd /tmp && \
    curl -fsSL -o ft.rpm "${VAULT4}/freetype-2.1.9-6.el4.x86_64.rpm" && \
    curl -fsSL -o vf.rpm "${VAULT4}/VFlib2-2.25.6-25.x86_64.rpm" && \
    curl -fsSL -o glib.rpm "${VAULT3}/glib-1.2.10-11.1.x86_64.rpm" && \
    curl -fsSL -o gp.rpm "${VAULT3}/gimp-print-4.2.4-5.1.x86_64.rpm" && \
    curl -fsSL -o gs.rpm "${GS}" && \
    mkdir -p /tmp/ft && (cd /tmp/ft && rpm2cpio /tmp/ft.rpm | cpio -idm --quiet) && \
    cp -a /tmp/ft/usr/lib64/libttf.so.2* /usr/lib64/ && \
    cd / && rpm2cpio /tmp/vf.rpm | cpio -idm --quiet && \
    cd / && rpm2cpio /tmp/glib.rpm | cpio -idm --quiet && \
    cd / && rpm2cpio /tmp/gp.rpm | cpio -idm --quiet && \
    cd / && rpm2cpio /tmp/gs.rpm | cpio -idm --quiet && \
    ldconfig && \
    rm -rf /tmp/*.rpm /tmp/ft

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
