FROM ubuntu:17.10

# Artful is EOL: serve packages from old-releases.ubuntu.com and skip the expired
# Valid-Until check. universe is included so the debug build can later pull
# elfutils (ghostscript-dbg is in main).
RUN echo "deb http://old-releases.ubuntu.com/ubuntu artful main universe" > /etc/apt/sources.list && \
    apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --no-install-recommends binutils gdb ghostscript && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
