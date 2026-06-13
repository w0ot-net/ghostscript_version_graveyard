FROM ubuntu:22.04

ARG GS_VERSION=9.18
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

RUN curl -fSL "https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs918/ghostscript-${GS_VERSION}.tar.gz" \
        -o /tmp/gs.tar.gz && \
    tar xzf /tmp/gs.tar.gz -C /tmp && \
    cd /tmp/ghostscript-${GS_VERSION} && \
    ./configure --prefix=/opt/gs && \
    make -j$(nproc) && \
    make install && \
    rm -rf /tmp/gs.tar.gz /tmp/ghostscript-${GS_VERSION}

ENV PATH="/opt/gs/bin:${PATH}"

WORKDIR /work
ENTRYPOINT ["gs"]
