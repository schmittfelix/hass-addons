# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM=ghcr.io/hassio-addons/base:15.0.7
FROM $BUILD_FROM AS shairport-sync-build

# Install build dependencies
RUN apk add --update \
    build-base \
    autoconf \
    automake \
    libtool \
    pkgconfig \
    cmake \
    git \
    soxr-dev \
    avahi-dev \
    alsa-lib-dev \
    popt-dev \
    libconfig-dev \
    openssl-dev \
    libvorbis-dev \
    opus-dev \
    flac-dev \
    libsndfile-dev

##### ALAC #####
RUN git clone https://github.com/mikebrady/alac
WORKDIR /alac
RUN autoreconf -fi \
    && ./configure \
    && make \
    && make install DESTDIR=/alac-install
WORKDIR /
##### ALAC END #####

##### NQPTP #####
RUN git clone --branch development https://github.com/mikebrady/nqptp
WORKDIR /nqptp
RUN autoreconf -fi \
    && ./configure \
    && make \
    && make install DESTDIR=/nqptp-install
WORKDIR /
##### NQPTP END #####


##### Shairport-sync ######
RUN git clone --branch development https://github.com/mikebrady/shairport-sync
WORKDIR /shairport-sync
RUN autoreconf -fi \
    && ./configure \
    --with-airplay-2 \
    --with-alsa \
    --with-pa \
    --with-stdout \
    --with-pipe \
    --with-soxr \
    --with-apple-alac \
    --with-convolution \
    --with-metadata \
    --with-mqtt-client \
    --with-dbus-interface \
    --with-mpris-interface \
    --sysconfdir=/etc \
    --with-ssl=openssl \
    --with-avahi \
    --with-external-mdns
RUN make
RUN make install DESTDIR=/shairport-sync-install
##### Shairpoint-sync END #####

# Run image
FROM $BUILD_FROM
COPY --from=shairport-sync-build /shairport-sync-install /
COPY --from=shairport-sync-build /alac-install /
COPY --from=shairport-sync-build /nqptp-install /

# for avahi-config and snapcast service
COPY rootfs /

# Install runtime dependencies
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories && apk update
RUN apk add --update \
    popt \
    soxr \
    libconfig \
    libvorbis \
    opus \
    flac \
    alsa-lib \
    alsa-plugins-pulse \
    libgcc \
    libstdc++ \
    expat \
    avahi-libs \
    dbus \
    avahi \
    nss \
    avahi-compat-libdns_sd \
    pulseaudio \
    snapcast \
    librespot

EXPOSE 1780
EXPOSE 1705
EXPOSE 1704

RUN for file in /etc/services.d/*/*; do chmod a+xwr $file; done
# RUN for file in /etc/cont-init.d/*; do chmod a+xwr $file; done

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