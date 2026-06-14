FROM debian:bookworm

RUN apt-get update && \
    apt-get install -y --no-install-recommends binutils gdb ghostscript && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
