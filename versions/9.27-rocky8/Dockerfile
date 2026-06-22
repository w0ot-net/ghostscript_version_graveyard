FROM rockylinux/rockylinux:8

# Rocky Linux 8 is an independent RHEL rebuild shipping the EL8 ghostscript 9.27
# (the same upstream version this repo also builds from Debian buster, but a
# different distro's patch set and toolchain).
RUN dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
