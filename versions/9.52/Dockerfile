FROM opensuse/leap:15.5

RUN zypper --non-interactive refresh && \
    zypper --non-interactive install --no-recommends ghostscript && \
    zypper clean --all

WORKDIR /work
ENTRYPOINT ["gs"]
