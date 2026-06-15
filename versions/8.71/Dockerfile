FROM ubuntu:18.04

ARG GS_VERSION=8.71
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        autoconf \
        automake \
        build-essential \
        ca-certificates \
        curl \
        libxext-dev \
        libx11-dev \
        libxt-dev \
        && rm -rf /var/lib/apt/lists/*

RUN curl -fSL "https://github.com/ArtifexSoftware/ghostpdl/archive/refs/tags/ghostscript-${GS_VERSION}.tar.gz" \
        -o /tmp/gs.tar.gz && \
    tar xzf /tmp/gs.tar.gz -C /tmp && \
    cd /tmp/ghostpdl-ghostscript-${GS_VERSION}/gs && \
    sed -i '/^struct timeval {$/,/^};$/d' base/time_.h && \
    cp /usr/share/automake-*/config.guess /usr/share/automake-*/config.sub . && \
    ./autogen.sh --prefix=/opt/gs && \
    make -j1 && \
    make install && \
    rm -rf /tmp/gs.tar.gz /tmp/ghostpdl-ghostscript-${GS_VERSION}

RUN apt-get update && \
    apt-get install -y --no-install-recommends gdb && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="/opt/gs/bin:${PATH}"

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
