# Base image updated by Renovate
ARG BUILD_FROM=ghcr.io/matter-js/matterjs-server:1.4.0@sha256:54232d0d3e7dff5a54759469d2753399270412b4c30c55b31750a4595e4cb236
FROM ${BUILD_FROM}

# The matter.js base image runs as an unprivileged user; the app requires root
# for s6-overlay and for Bluetooth via Noble raw HCI access.
# hadolint ignore=DL3002
USER root

# Default ENV
ENV \
    LOG_LEVEL="" \
    LANG="C.UTF-8" \
    DEBIAN_FRONTEND="noninteractive" \
    CURL_CA_BUNDLE="/etc/ssl/certs/ca-certificates.crt" \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_CMD_WAIT_FOR_SERVICES=1 \
    S6_SERVICES_READYTIME=50

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Build Args
ARG \
    BASHIO_VERSION=0.19.0 \
    TEMPIO_VERSION=2024.11.2 \
    S6_OVERLAY_VERSION=3.1.6.2

# Base system
WORKDIR /usr/src
ARG TARGETARCH

RUN \
    set -x \
    && apt-get update && apt-get install -y --no-install-recommends \
        bash \
        jq \
        tzdata \
        curl \
        ca-certificates \
        xz-utils \
        iproute2 \
    && mkdir -p /usr/share/man/man1 \
    \
    && if [ -z "${TARGETARCH}" ]; then \
        echo "TARGETARCH is not set, please use Docker BuildKit for the build." && exit 1; \
    fi \
    && case "${TARGETARCH}" in \
        amd64) \
            S6_OVERLAY_ARCH="x86_64" \
            TEMPIO_ARCH="amd64" \
            ;; \
        arm64) \
            S6_OVERLAY_ARCH="aarch64" \
            TEMPIO_ARCH="aarch64" \
            ;; \
        *) echo "Unsupported TARGETARCH: ${TARGETARCH}" && exit 1 ;; \
    esac \
    \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_OVERLAY_ARCH}.tar.xz" \
        | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" \
        | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-arch.tar.xz" \
        | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-noarch.tar.xz" \
        | tar Jxvf - -C / \
    && mkdir -p /etc/fix-attrs.d \
    && mkdir -p /etc/services.d \
    \
    && curl -L -f -s -o /usr/bin/tempio \
        "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${TEMPIO_ARCH}" \
    && chmod a+x /usr/bin/tempio \
    \
    && mkdir -p /usr/src/bashio \
    && curl -L -f -s "https://github.com/hassio-addons/bashio/archive/v${BASHIO_VERSION}.tar.gz" \
        | tar -xzf - --strip 1 -C /usr/src/bashio \
    && mv /usr/src/bashio/lib /usr/lib/bashio \
    && ln -s /usr/lib/bashio/bashio /usr/bin/bashio \
    \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /usr/src/*

# S6-Overlay
WORKDIR /root
ENTRYPOINT ["/init"]

COPY rootfs /

LABEL \
    org.opencontainers.image.title="Home Assistant App: Matter Server" \
    org.opencontainers.image.description="Matter WebSocket Server for Home Assistant Matter support." \
    org.opencontainers.image.source="https://github.com/home-assistant/app-matter-server" \
    org.opencontainers.image.licenses="Apache License 2.0"
