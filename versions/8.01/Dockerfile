# Ghostscript 8.01 from the Ubuntu 4.10 "warty" old-releases archive (gs-gpl deb).
# Warty has no usable base image, so the amd64 gs-gpl deb is extracted onto the
# Debian etch archive base, which still ships the old shared-library sonames the
# 2004 binary needs (libpng12, libjpeg62, libgimpprint1, libpaper1, X libs).
FROM debian/eol:etch

ARG WARTY=http://old-releases.ubuntu.com/ubuntu/pool/main

RUN apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --force-yes \
        binutils gdb wget \
        libpaper1 libxt6 libsm6 libice6 libxext6 libx11-6 \
        libpng12-0 libjpeg62 zlib1g && \
    cd /tmp && \
    wget -q "${WARTY}/g/gimp-print/libgimpprint1_4.2.6-5ubuntu2_amd64.deb" && \
    wget -q "${WARTY}/g/gs-gpl/gs-gpl_8.01-4_amd64.deb" && \
    dpkg-deb -x libgimpprint1_4.2.6-5ubuntu2_amd64.deb / && \
    dpkg-deb -x gs-gpl_8.01-4_amd64.deb / && \
    ldconfig && \
    ln -sf /usr/bin/gs-gpl /usr/bin/gs && \
    rm -f /tmp/*.deb && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
