# Ghostscript 7.07 from the CentOS 4.6 vault updates (ghostscript x86_64 RPM).
# CentOS 4 has no usable base image, so the gs binary and its libgs.so.7 are
# extracted onto a CentOS 6 base (glibc 2.12). CentOS 6 supplies most runtime
# sonames, but two extinct font-engine libraries from the 2003 build are pulled
# from the CentOS 4 vault: libVFlib2.so.24 (VFlib2) and libttf.so.2 (FreeType 1,
# which the el4 freetype package still bundled alongside libfreetype.so.6).
# urw-fonts and ghostscript-fonts are required or gs aborts at startup.
FROM centos:6

ARG VAULT=https://ftp.iij.ad.jp/pub/linux/centos-vault/4.6/os/x86_64/CentOS/RPMS
ARG GS=https://ftp.iij.ad.jp/pub/linux/centos-vault/4.6/updates/x86_64/RPMS/ghostscript-7.07-33.2.el4_6.1.x86_64.rpm

RUN sed -i 's|^mirrorlist=|#mirrorlist=|; s|^#baseurl=http://mirror.centos.org|baseurl=https://vault.centos.org|' /etc/yum.repos.d/CentOS-Base.repo && \
    yum -y install libpng libjpeg-turbo cups-libs glib2 libstdc++ zlib \
        libX11 libXext libXt libSM libICE urw-fonts ghostscript-fonts && \
    yum clean all && \
    cd /tmp && \
    curl -fsSL -o ft.rpm "${VAULT}/freetype-2.1.9-6.el4.x86_64.rpm" && \
    curl -fsSL -o vf.rpm "${VAULT}/VFlib2-2.25.6-25.x86_64.rpm" && \
    curl -fsSL -o gs.rpm "${GS}" && \
    mkdir -p /tmp/ft && (cd /tmp/ft && rpm2cpio /tmp/ft.rpm | cpio -idm --quiet) && \
    cp -a /tmp/ft/usr/lib64/libttf.so.2* /usr/lib64/ && \
    cd / && rpm2cpio /tmp/vf.rpm | cpio -idm --quiet && \
    cd / && rpm2cpio /tmp/gs.rpm | cpio -idm --quiet && \
    ldconfig && \
    rm -rf /tmp/*.rpm /tmp/ft

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
