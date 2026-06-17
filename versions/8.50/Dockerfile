# Ghostscript 8.50 from the Ubuntu 6.10 "edgy" old-releases archive (gs-gpl deb).
# Edgy has no usable base image. The 2006 binary needs libc6 >= 2.4, which the
# Debian etch archive base (glibc 2.3.6) is too old to satisfy, so the amd64
# gs-gpl deb is extracted onto the Debian lenny archive base (glibc 2.7), which
# still ships the old shared-library sonames the binary needs (libpaper1, X libs).
FROM debian/eol:lenny

ARG EDGY=http://old-releases.ubuntu.com/ubuntu/pool/main

RUN apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --force-yes \
        binutils gdb wget \
        libpaper1 libxt6 libsm6 libice6 libxext6 libx11-6 && \
    cd /tmp && \
    wget -q "${EDGY}/g/gs-gpl/gs-gpl_8.50-1.1ubuntu1_amd64.deb" && \
    dpkg-deb -x gs-gpl_8.50-1.1ubuntu1_amd64.deb / && \
    ldconfig && \
    ln -sf /usr/bin/gs-gpl /usr/bin/gs && \
    rm -f /tmp/*.deb && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
