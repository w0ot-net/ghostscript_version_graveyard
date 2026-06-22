FROM amazonlinux:2

# Amazon Linux 2 ships ghostscript 9.54.0 (the same upstream version this repo
# also builds from Fedora 35, but Amazon's older yum-based toolchain and patch
# set — a distinct build from the Amazon Linux 2023 image).
RUN yum install -y binutils gdb ghostscript && \
    yum clean all

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
