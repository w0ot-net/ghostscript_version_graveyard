FROM archlinux:base

# Pin pacman to the Arch Linux archive snapshot that shipped ghostscript 10.04.0
# (the extra.db lists 10.04.0 from 2024/09/19), downgrade the base system to that
# snapshot, then install gs. --overwrite resolves split-package file conflicts
# between the current base image and the older snapshot. SigLevel=Never because
# archived packages may be signed by keys no longer in the current keyring.
RUN printf 'Server=https://archive.archlinux.org/repos/2024/09/19/$repo/os/$arch\n' > /etc/pacman.d/mirrorlist && \
    sed -i 's/^SigLevel.*/SigLevel = Never/' /etc/pacman.conf && \
    pacman -Syyuu --noconfirm --overwrite '*' && \
    pacman -S --noconfirm --needed --overwrite '*' ghostscript gdb binutils && \
    pacman -Scc --noconfirm

LABEL org.opencontainers.image.architecture="amd64" \
      net.w0ot.ghostscript.cpu-architecture="x86_64"

WORKDIR /work
ENTRYPOINT ["gs"]
