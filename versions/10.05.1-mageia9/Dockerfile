FROM mageia:9

# Mageia 9 ships ghostscript 10.05.1 (the same upstream version this repo also
# builds from Debian trixie, but Mageia's independent Mandriva-lineage toolchain
# and patch set).
RUN dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
