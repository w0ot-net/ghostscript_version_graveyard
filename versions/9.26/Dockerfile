FROM ubuntu:22.04

ARG GS_VERSION=9.26
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        autoconf \
        automake \
        build-essential \
        ca-certificates \
        curl \
        libx11-dev \
        libxext-dev \
        libxt-dev \
        && rm -rf /var/lib/apt/lists/*

RUN curl -fSL "https://github.com/ArtifexSoftware/ghostpdl/archive/refs/tags/ghostscript-${GS_VERSION}.tar.gz" \
        -o /tmp/gs.tar.gz && \
    tar xzf /tmp/gs.tar.gz -C /tmp && \
    cd /tmp/ghostpdl-ghostscript-${GS_VERSION} && \
    cp /usr/share/automake-*/config.guess /usr/share/automake-*/config.sub . && \
    ./autogen.sh --prefix=/opt/gs && \
    make -j$(nproc) && \
    make install && \
    rm -rf /tmp/gs.tar.gz /tmp/ghostpdl-ghostscript-${GS_VERSION}

ENV PATH="/opt/gs/bin:${PATH}"

WORKDIR /work
ENTRYPOINT ["gs"]
