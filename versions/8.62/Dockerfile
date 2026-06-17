# Ghostscript 8.62 from the Fedora 9 archive (ghostscript x86_64 RPM).
# Fedora 9 has no usable base image, so the gs binary and its libgs.so.8 are
# extracted onto a CentOS 6 base (glibc 2.12), which still provides every old
# runtime soname the 2008 build needs (libjasper.so.1, libtiff.so.3,
# libjpeg.so.62, libpng12.so.0, libcups*, krb5, libfontconfig.so.1).
FROM centos:6

ARG RPM=https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/9/Everything/x86_64/os/Packages/ghostscript-8.62-3.fc9.x86_64.rpm

RUN sed -i 's|^mirrorlist=|#mirrorlist=|; s|^#baseurl=http://mirror.centos.org|baseurl=https://vault.centos.org|' /etc/yum.repos.d/CentOS-Base.repo && \
    yum -y install libtiff libjpeg-turbo libpng cups-libs krb5-libs fontconfig libstdc++ jasper-libs && \
    yum clean all && \
    curl -fsSL -o /tmp/gs.rpm "${RPM}" && \
    cd / && rpm2cpio /tmp/gs.rpm | cpio -idm --quiet && \
    ldconfig && \
    rm -f /tmp/gs.rpm

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
