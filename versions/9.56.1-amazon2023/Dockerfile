FROM amazonlinux:2023

# Amazon Linux 2023 ships ghostscript 9.56.1 (the same upstream version this repo
# also builds from Fedora 36, but Amazon's own toolchain, glibc, and patch set).
RUN dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
