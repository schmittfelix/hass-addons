# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM=ghcr.io/hassio-addons/base:15.0.7
FROM $BUILD_FROM AS build-base
# Common tools
RUN apk add --update build-base autoconf automake libtool pkgconfig cmake 
# Common dependencies
RUN apk add --update soxr-dev avahi-dev alsa-lib-dev

# Shairport-sync build image
FROM build-base AS shairport-sync-build
RUN apk add --update popt-dev libconfig-dev openssl-dev
# ALAC dependency
WORKDIR /alac
COPY alac .
RUN autoreconf -fi && ./configure
RUN make && make install && make install DESTDIR=/alac-install
# Shairport-sync
WORKDIR /shairport-sync
COPY shairport-sync .
RUN autoreconf -fi && ./configure --sysconfdir=/etc --with-soxr --with-avahi --with-ssl=openssl --with-metadata --with-apple-alac -with-stdout
RUN make && make install DESTDIR=/shairport-sync-install

# Run image
FROM $BUILD_FROM
COPY --from=shairport-sync-build /shairport-sync-install /
COPY --from=shairport-sync-build /alac-install /
# for avahi-config and snapcast service
COPY rootfs /
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && apk update
RUN apk add --update tini popt soxr libconfig libvorbis opus flac alsa-lib alsa-plugins-pulse libgcc libstdc++ expat avahi-libs dbus avahi nss avahi-compat-libdns_sd pulseaudio snapcast librespot

EXPOSE 1780
EXPOSE 1705 
EXPOSE 1704
RUN for file in /etc/services.d/*/*; do chmod a+xwr $file; done
RUN chmod a+x /snapcast-script
CMD ["/bin/sh", "-c", "/snapcast-script"]

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="schmittfelix" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="schmittfelix's addons" \
    org.opencontainers.image.authors="schmittfelix" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}