# gs 10.01.1 was a one-day Arch point release (2023/03/27-28). Downgrading a
# current Arch base three years in place gets throttled by archive.archlinux.org
# (hundreds of package fetches), so base on the era bootstrap rootfs (a coherent
# 2023 Arch userland) and install gs from the pinned 2023/03/28 archive snapshot.
# http + SigLevel=Never avoid old-CA TLS failures and an uninitialised keyring.
FROM debian:bookworm AS rootfs
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*
RUN curl -fSL "https://archive.archlinux.org/iso/2023.03.01/archlinux-bootstrap-2023.03.01-x86_64.tar.gz" -o /tmp/b.tar.gz && \
    mkdir /out && tar xzf /tmp/b.tar.gz -C /out --numeric-owner

FROM scratch
COPY --from=rootfs /out/root.x86_64/ /
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin
SHELL ["/bin/bash", "-c"]

# Write the pacman config with printf (the bootstrap is minimal) and explicitly
# pull the coreutils/grep/gawk userland the combined-image clone stage needs.
RUN printf '%s\n' '[options]' 'Architecture = auto' 'SigLevel = Never' \
        '[core]' 'Include = /etc/pacman.d/mirrorlist' \
        '[extra]' 'Include = /etc/pacman.d/mirrorlist' \
        '[community]' 'Include = /etc/pacman.d/mirrorlist' > /etc/pacman.conf && \
    printf 'Server = http://archive.archlinux.org/repos/2023/03/28/$repo/os/$arch\n' > /etc/pacman.d/mirrorlist && \
    pacman -Syy --noconfirm && \
    pacman -S --noconfirm --needed coreutils grep gawk sed ghostscript gdb binutils && \
    (pacman -Scc --noconfirm || true)

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
