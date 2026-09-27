ARG DISTRO_VERSION="26.04"
ARG DISTRO="ubuntu"
FROM ${DISTRO}:${DISTRO_VERSION}

ARG DISTRO_VERSION
ENV DISTRO_VERSION=${DISTRO_VERSION}
ARG DISTRO
ENV DISTRO=${DISTRO}

# APT mirror used inside the image. Leave empty to use the base image's own
# sources. Handles Ubuntu (archive/ports/security) and Debian (deb.debian.org)
# deb822 sources, as well as a classic sources.list. Example:
#   --build-arg APT_MIRROR=http://mirrors.aliyun.com/debian \
#   --build-arg APT_SECURITY_MIRROR=http://mirrors.aliyun.com/debian-security
ARG APT_MIRROR=""
ARG APT_SECURITY_MIRROR=""

# Install Dependencies
RUN set -eux; \
    if [ -n "$APT_MIRROR" ]; then \
        for f in /etc/apt/sources.list /etc/apt/sources.list.d/*.sources; do \
            [ -e "$f" ] || continue; \
            # Security host first: on Debian it shares a prefix with the archive
            # host, and running the archive sed first would mangle it.
            if [ -n "$APT_SECURITY_MIRROR" ]; then \
                sed -i -E "s#https?://(deb\.debian\.org/debian-security|security\.debian\.org/debian-security|security\.ubuntu\.com/ubuntu)#${APT_SECURITY_MIRROR}#g" "$f"; \
            fi; \
            sed -i -E "s#https?://(deb\.debian\.org/debian|archive\.ubuntu\.com/ubuntu|ports\.ubuntu\.com/ubuntu-ports)#${APT_MIRROR}#g" "$f"; \
        done; \
        grep -RhsE "^(URIs|deb)" /etc/apt/sources.list.d/ /etc/apt/sources.list 2>/dev/null || true; \
    fi; \
    export DEBIAN_FRONTEND="noninteractive"; \
    apt-get -qq update && \
    apt-get -qq -y --no-install-recommends install \
    # Build Tools
    build-essential \
    debhelper \
    devscripts \
    fakeroot \
    libbz2-dev \
    libonig-dev \
    lintian \
    lsb-release \
    minisign \
    pandoc \
    quilt \
    wget \
    # Ghostty Dependencies
    # https://ghostty.org/docs/install/build#debian-and-ubuntu
    gettext \
    libadwaita-1-dev \
    libgtk-4-dev \
    libxml2-utils && \
    # Install libgtk4-layer-shell-dev but not on Debian
    if [ "$DISTRO" = "ubuntu" ]; then \
        apt-get -qq -y --no-install-recommends install libgtk4-layer-shell-dev; \
    fi && \
    # Clean up for better caching
    rm -rf /var/lib/apt/lists/*

# Install zig
# https://ziglang.org/download/
# ZIG_BASE_URL may point at a mirror of ziglang.org/download. The tarball is
# still verified with minisign below, so a mirror can only ever serve the
# genuine artifact.
ARG ZIG_VERSION="0.14.1"
ARG ZIG_BASE_URL="https://ziglang.org/download"
RUN set -eux; \
    arch="$(uname -m)"; \
    tarball="zig-$arch-linux-$ZIG_VERSION.tar.xz"; \
    url="$ZIG_BASE_URL/$ZIG_VERSION/$tarball"; \
    echo "$url"; \
    wget -q "$url" "$url.minisig"; \
    minisign -Vm "$tarball" -P "RWSGOq2NVecA2UPNdBUZykf1CCb147pkmdtYxgb3Ti+JO/wCYvhbAb/U"; \
    tar -xf "$tarball" -C /opt; \
    rm "$tarball" "$tarball.minisig"; \
    ln -s "/opt/zig-$arch-linux-$ZIG_VERSION/zig" /usr/local/bin/zig
