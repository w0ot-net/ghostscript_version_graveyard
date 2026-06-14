FROM debian:jessie

RUN echo "deb http://archive.debian.org/debian jessie main" > /etc/apt/sources.list && \
    apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --force-yes --no-install-recommends ghostscript && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /work
ENTRYPOINT ["gs"]
