FROM opensuse/leap:15.0

# Leap 15.0 is EOL but its repos remain hosted on download.opensuse.org. Disable
# the update repos so the GA ghostscript 9.23 is installed (the update pocket
# ships 9.27).
RUN zypper mr --disable repo-update repo-update-non-oss || true && \
    zypper --non-interactive --no-gpg-checks refresh && \
    zypper --non-interactive --no-gpg-checks install --no-recommends binutils gdb ghostscript && \
    zypper clean --all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
