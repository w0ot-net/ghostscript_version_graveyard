FROM opensuse/leap:15.5

RUN zypper mr --disable repo-update repo-update-non-oss repo-backports-update repo-sle-update || true && \
    zypper --non-interactive --no-gpg-checks refresh && \
    zypper --non-interactive --no-gpg-checks install --no-recommends binutils gdb ghostscript && \
    zypper clean --all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
