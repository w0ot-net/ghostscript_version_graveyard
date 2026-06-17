# Ghostscript 8.54 from the Debian 4.0 "etch" archive (gs-gpl distro package).
# The hosted gs binary is the amd64 gs-gpl build shipped by Debian etch.
FROM debian/eol:etch

RUN apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --force-yes binutils gdb gs-gpl gs-common && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
