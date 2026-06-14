FROM opensuse/leap:15.5

RUN zypper --non-interactive refresh && \
    zypper --non-interactive install --no-recommends ghostscript && \
    zypper clean --all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
