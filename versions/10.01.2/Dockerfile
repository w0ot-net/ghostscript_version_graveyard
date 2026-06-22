FROM fedora:39

# Fedora 39 is EOL: repoint every repo at the archive (newer Fedora ships a
# download.example placeholder baseurl, so match any host), and disable updates
# so the GA ghostscript 10.01.2 is installed.
RUN sed -i -e 's|^metalink=|#metalink=|' \
        -e 's|^#\?baseurl=https\?://[^/]*/pub/fedora/linux|baseurl=https://archives.fedoraproject.org/pub/archive/fedora/linux|' \
        /etc/yum.repos.d/*.repo && \
    sed -i 's|^enabled=1|enabled=0|' /etc/yum.repos.d/fedora-updates*.repo && \
    dnf install -y --setopt=install_weak_deps=False binutils gdb ghostscript && \
    dnf clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
