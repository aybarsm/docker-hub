# syntax=docker/dockerfile:1

FROM debian:bookworm-slim AS build

ARG VER_PHPEXTBROTLI='0.18.0'

WORKDIR /tmp

RUN set -eux \
    && . /etc/os-release \
    && DEBIAN_FRONTEND=noninteractive apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --no-install-suggests -y \
        ca-certificates \
        curl \
        build-essential \
        libbrotli-dev \
        git \
    && curl https://packages.sury.org/debsuryorg-archive-keyring.deb --output /tmp/debsuryorg-archive-keyring.deb \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --no-install-suggests -y /tmp/debsuryorg-archive-keyring.deb \
    && echo "deb [signed-by=/usr/share/keyrings/debsuryorg-archive-keyring.gpg] https://packages.sury.org/php/ $VERSION_CODENAME main" | tee /etc/apt/sources.list.d/php-sury.list >/dev/null \
    && DEBIAN_FRONTEND=noninteractive apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --no-install-suggests -y \
        php8.4-dev \
    && git clone --recursive --depth=1 -b ${VER_PHPEXTBROTLI} --single-branch https://github.com/kjdev/php-ext-brotli.git /tmp/php-ext-brotli \
    && cd /tmp/php-ext-brotli \
    && phpize \
    && ./configure --with-libbrotli \
    && make \
    && make install \
    && PATH_PHPCONFIGBIN=$(readlink -f $(which php-config)) \
    && PATH_PHPEXTDIR=$("${PATH_PHPCONFIGBIN}" --extension-dir) \
    && cp "${PATH_PHPEXTDIR}/brotli.so" /tmp/brotli.so \
    && cp "${PATH_PHPCONFIGBIN}" /tmp/php-config

FROM debian:bookworm-slim
LABEL maintainer="aybarsm" tag_build="$BUILD_DATE"

ARG VER_S6OVERLAY='3.2.1.0'
ARG VER_NODEJS='22'

WORKDIR /tmp

RUN set -eux \
    && DEBIAN_FRONTEND=noninteractive apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --no-install-suggests -y \
        ca-certificates \
        curl \
        libcap2-bin \
        xz-utils \
        libbrotli1

COPY --from=build /usr/share/keyrings/debsuryorg-archive-keyring.gpg /usr/share/keyrings/debsuryorg-archive-keyring.gpg
COPY --from=build /etc/apt/sources.list.d/php-sury.list /etc/apt/sources.list.d/php-sury.list
COPY --from=build /tmp/php-config /usr/local/bin/php-config
COPY --from=build /tmp/brotli.so /tmp/brotli.so

RUN set -eux \
    && ARCH=$(uname -m) \
    && curl -sLS https://deb.nodesource.com/setup_${VER_NODEJS}.x | bash - \
    && DEBIAN_FRONTEND=noninteractive apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --no-install-suggests -y \
		php8.4-cli \
        php8.4-bcmath \
        php8.4-bz2 \
        php8.4-curl \
        php8.4-decimal \
        php8.4-gd \
        php8.4-gmp \
        php8.4-grpc \
        php8.4-http \
        php8.4-igbinary \
        php8.4-imagick \
        php8.4-imap \
        php8.4-inotify \
        php8.4-intl \
        php8.4-lz4 \
        php8.4-mailparse \
        php8.4-maxminddb \
        php8.4-mbstring \
        php8.4-mcrypt \
        php8.4-mongodb \
        php8.4-msgpack \
        php8.4-mysql \
        php8.4-opentelemetry \
        php8.4-pgsql \
        php8.4-pq \
        php8.4-protobuf \
        php8.4-psr \
        php8.4-rdkafka \
        php8.4-readline \
        php8.4-redis \
        php8.4-rrd \
        php8.4-snmp \
        php8.4-sqlite3 \
        php8.4-ssh2 \
        php8.4-swoole \
        php8.4-uploadprogress \
        php8.4-uuid \
        php8.4-xml \
        php8.4-xmlrpc \
        php8.4-yaml \
        php8.4-zip \
        php8.4-zmq \
        php8.4-zst \
        nodejs \
    && setcap "cap_net_bind_service=+ep" $(readlink -f $(which php)) \
    && setcap "cap_net_bind_service=+ep" $(readlink -f $(which node)) \
    && chmod +x /usr/local/bin/php-config \
    && PATH_PHPEXTDIR=$(/usr/local/bin/php-config --extension-dir) \
    && PATH_PHPINIDIR=$(/usr/local/bin/php-config --ini-dir) \
    && cp /tmp/brotli.so "${PATH_PHPEXTDIR}/brotli.so" \
    && echo "extension=brotli.so" | tee "${PATH_PHPINIDIR}/20-brotli.ini" >/dev/null \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/local/bin --filename=composer \
    && chmod +x /usr/local/bin/composer \
    && /usr/local/bin/composer self-update --stable \
    && npm install -g npm@latest \
    && curl -sfL https://github.com/just-containers/s6-overlay/releases/download/v${VER_S6OVERLAY}/s6-overlay-noarch.tar.xz -o /tmp/s6-overlay-noarch.tar.xz \
    && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
    && curl -sfL https://github.com/just-containers/s6-overlay/releases/download/v${VER_S6OVERLAY}/s6-overlay-${ARCH}.tar.xz -o /tmp/s6-overlay-arch.tar.xz \
    && tar -C / -Jxpf /tmp/s6-overlay-arch.tar.xz \
    && DEBIAN_FRONTEND=noninteractive apt-get clean \
    && DEBIAN_FRONTEND=noninteractive apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /tmp/* /var/tmp/* /usr/share/doc/* /var/lib/{apt,dpkg,cache,log}/

WORKDIR /etc/s6-overlay/s6-rc.d

ENTRYPOINT ["/init"]