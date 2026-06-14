FROM fedora:20

RUN sed -i 's|^metalink=|#metalink=|; s|^#baseurl=http://download.fedoraproject.org/pub/fedora/linux|baseurl=https://archives.fedoraproject.org/pub/archive/fedora/linux|' /etc/yum.repos.d/fedora.repo && \
    sed -i 's|^enabled=1|enabled=0|' /etc/yum.repos.d/fedora-updates*.repo && \
    yum install -y ghostscript && \
    yum clean all

WORKDIR /work
ENTRYPOINT ["gs"]
