FROM fedora:21

RUN sed -i 's|^metalink=|#metalink=|; s|^#baseurl=http://download.fedoraproject.org/pub/fedora/linux|baseurl=https://archives.fedoraproject.org/pub/archive/fedora/linux|' /etc/yum.repos.d/fedora.repo && \
    sed -i 's|^enabled=1|enabled=0|' /etc/yum.repos.d/fedora-updates*.repo && \
    yum install -y ghostscript && \
    yum clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
