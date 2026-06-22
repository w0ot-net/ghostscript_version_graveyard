# gs 9.15 only ever shipped in Ubuntu 15.04 (vivid), whose base image uses a
# legacy v1 manifest that current Docker/containerd refuses to pull. Instead,
# unpack vivid's own ghostscript/libgs9 debs onto ubuntu:16.04 (xenial), whose
# newer glibc runs the older binary and supplies ABI-compatible support libs.
# The gs binary and libgs.so.9 are the real vivid 9.15 build.
FROM ubuntu:16.04

RUN echo "deb http://archive.ubuntu.com/ubuntu xenial main universe" > /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        binutils gdb curl ca-certificates \
        libfontconfig1 libfreetype6 libjbig2dec0 libjpeg-turbo8 liblcms2-2 \
        libpaper1 libpng12-0 libtiff5 zlib1g libidn11 libcups2 libcupsimage2 \
        libijs-0.35 fonts-droid-fallback && \
    cd /tmp && \
    base=http://old-releases.ubuntu.com/ubuntu/pool/main/g/ghostscript && \
    for p in ghostscript_9.15+dfsg-0ubuntu2.1_amd64.deb \
             libgs9_9.15+dfsg-0ubuntu2.1_amd64.deb \
             libgs9-common_9.15+dfsg-0ubuntu2.1_all.deb; do \
        curl -fSLO "$base/$p"; \
    done && \
    for d in *.deb; do dpkg -x "$d" /; done && \
    ldconfig && \
    rm -f /tmp/*.deb && \
    rm -rf /var/lib/apt/lists/* && \
    gs --version

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
