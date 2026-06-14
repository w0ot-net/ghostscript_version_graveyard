FROM ubuntu:18.04

RUN echo "deb http://archive.ubuntu.com/ubuntu bionic main" > /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends ghostscript && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /work
ENTRYPOINT ["gs"]
