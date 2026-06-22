FROM debian:bullseye

# gs 9.53.2 was only in Debian unstable for a few weeks. Pin apt to the
# snapshot.debian.org timestamp that carried it; the snapshot Release files are
# long expired, so disable the Valid-Until check.

# Install the debugger tooling from the base distro first; installing gdb together
# with the snapshot's ghostscript triggers apt dependency conflicts.
RUN apt-get update && \
    apt-get install -y --no-install-recommends binutils gdb elfutils file curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*

RUN printf 'deb [check-valid-until=no] http://snapshot.debian.org/archive/debian/20200925T205109Z/ sid main\n' \
        > /etc/apt/sources.list && \
    printf 'Acquire::Check-Valid-Until "false";\nAcquire::Retries "3";\n' \
        > /etc/apt/apt.conf.d/99snapshot && \
    apt-get update && \
    apt-get install -y --no-install-recommends ghostscript && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
