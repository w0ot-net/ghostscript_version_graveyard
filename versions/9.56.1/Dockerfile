FROM fedora:36

RUN dnf install -y --setopt=install_weak_deps=False ghostscript && \
    dnf clean all

WORKDIR /work
ENTRYPOINT ["gs"]
