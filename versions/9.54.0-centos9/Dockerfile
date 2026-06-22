FROM quay.io/centos/centos:stream9

# CentOS Stream 9 is an independent rebuild of the EL9 ghostscript 9.54.0 (the
# same upstream version this repo also builds from Fedora 35, but a different
# distro's patch set and toolchain).
RUN dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
