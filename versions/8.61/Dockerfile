# Ghostscript 8.61 from the Ubuntu 7.10 "gutsy" old-releases archive.
# Gutsy split Ghostscript into a ghostscript front-end plus a libgs8 shared
# library, so the gs binary loads libgs.so.8 and its rendering dependencies.
# Gutsy has no usable base image and the 2007 binary needs libc6 >= 2.6, which
# the Debian etch archive (glibc 2.3.6) is too old to satisfy, so the debs are
# extracted onto the Debian lenny archive base (glibc 2.7). Lenny supplies every
# runtime soname except libgnutls.so.13, which is pulled from the gutsy archive.
FROM debian/eol:lenny

ARG GUTSY=http://old-releases.ubuntu.com/ubuntu/pool/main

RUN apt-get -o Acquire::Check-Valid-Until=false update && \
    apt-get install -y --force-yes \
        binutils gdb wget \
        libcupsimage2 libcups2 libtiff4 libjpeg62 libpng12-0 libpaper1 \
        zlib1g libstdc++6 libfontconfig1 libkrb53 libcomerr2 \
        libtasn1-3 libgpg-error0 libgcrypt11 && \
    cd /tmp && \
    wget -q "${GUTSY}/g/gnutls13/libgnutls13_1.6.3-1build1_amd64.deb" && \
    wget -q "${GUTSY}/g/ghostscript/libgs8_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb" && \
    wget -q "${GUTSY}/g/ghostscript/ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb" && \
    wget -q "${GUTSY}/g/gsfonts/gsfonts_8.11+urwcyr1.0.7~pre41-3_all.deb" && \
    dpkg-deb -x libgnutls13_1.6.3-1build1_amd64.deb / && \
    dpkg-deb -x libgs8_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb / && \
    dpkg-deb -x ghostscript_8.61.dfsg.1~svn8187-0ubuntu3_amd64.deb / && \
    dpkg-deb -x gsfonts_8.11+urwcyr1.0.7~pre41-3_all.deb / && \
    ldconfig && \
    rm -f /tmp/*.deb && \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
