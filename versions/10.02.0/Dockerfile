FROM debian:bookworm

# gs 10.02.0 was only briefly in Debian unstable before 10.02.1. Pin apt to the
# snapshot.debian.org timestamp that carried it. bookworm uses the deb822
# debian.sources, which would otherwise mask the pinned snapshot, so remove it.

# Install the debugger tooling from the base distro first (while the base repos
# are still configured); installing gdb together with the snapshot's ghostscript
# triggers apt dependency conflicts.
RUN apt-get update && \
    apt-get install -y --no-install-recommends binutils gdb elfutils file curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*

RUN rm -f /etc/apt/sources.list.d/debian.sources && \
    printf 'deb [check-valid-until=no] http://snapshot.debian.org/archive/debian/20230914T031011Z/ sid main\n' \
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
