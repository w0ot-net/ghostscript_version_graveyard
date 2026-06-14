FROM ubuntu:24.04

ARG GS_VERSION=10.03.0
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
        libx11-dev \
        libxext-dev \
        libxt-dev \
        && rm -rf /var/lib/apt/lists/*

RUN curl -fSL "https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs10030/ghostscript-${GS_VERSION}.tar.gz" \
        -o /tmp/gs.tar.gz && \
    tar xzf /tmp/gs.tar.gz -C /tmp && \
    cd /tmp/ghostscript-${GS_VERSION} && \
    ./configure --prefix=/opt/gs && \
    make -j$(nproc) && \
    make install && \
    rm -rf /tmp/gs.tar.gz /tmp/ghostscript-${GS_VERSION}

RUN apt-get update && \
    apt-get install -y --no-install-recommends gdb && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="/opt/gs/bin:${PATH}"

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
