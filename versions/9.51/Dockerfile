# gs 9.51 only ever shipped in Arch Linux around 2020. Downgrading a current
# Arch base image five years hits unresolvable package conflicts, so instead base
# on the era bootstrap rootfs (a coherent 2020 Arch userland) and install gs from
# the pinned 2020/03/14 archive snapshot. http + SigLevel=Never avoid old-CA TLS
# failures and an uninitialised keyring.
FROM debian:bookworm AS rootfs
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*
RUN curl -fSL "https://archive.archlinux.org/iso/2020.03.01/archlinux-bootstrap-2020.03.01-x86_64.tar.gz" -o /tmp/b.tar.gz && \
    mkdir /out && tar xzf /tmp/b.tar.gz -C /out --numeric-owner

FROM scratch
COPY --from=rootfs /out/root.x86_64/ /
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin
SHELL ["/bin/bash", "-c"]

# The bootstrap rootfs ships only bash + pacman, so write the config with printf
# (no sed) and explicitly pull the coreutils/grep/gawk userland that the later
# combined-image clone stage depends on.
RUN printf '%s\n' '[options]' 'Architecture = auto' 'SigLevel = Never' \
        '[core]' 'Include = /etc/pacman.d/mirrorlist' \
        '[extra]' 'Include = /etc/pacman.d/mirrorlist' \
        '[community]' 'Include = /etc/pacman.d/mirrorlist' > /etc/pacman.conf && \
    printf 'Server = http://archive.archlinux.org/repos/2020/03/14/$repo/os/$arch\n' > /etc/pacman.d/mirrorlist && \
    pacman -Syy --noconfirm && \
    pacman -S --noconfirm --needed coreutils grep gawk sed ghostscript gdb binutils && \
    (pacman -Scc --noconfirm || true)

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
