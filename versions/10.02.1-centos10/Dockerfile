FROM quay.io/centos/centos:stream10

# CentOS Stream 10 is an independent rebuild of the EL10 ghostscript 10.02.1 (the
# same upstream version this repo also builds from Ubuntu 24.04, but a different
# distro's patch set and toolchain).
RUN dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
