# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.1-cli-alpine3.17"
ARG COMPOSER_VERSION="2.5"
ARG ROADRUNNER_VERSION="2023.3.0"

# ------------------------------------------------- Composer Image -------------------------------------------------
FROM composer:${COMPOSER_VERSION} as composer

# ------------------------------------------------- RoadRunner Image -----------------------------------------------
FROM spiralscout/roadrunner:${ROADRUNNER_VERSION} as roadrunner

# ------------------------------------------------- Supervisord Image -----------------------------------------------
FROM litan1106/golang-supervisord as supervisord

# ==================================================================================================================
#                                                --- PHP EXTENIONS ---
# ==================================================================================================================
FROM php:${VARIANT} as php-ext

ARG PECL_PKGS="igbinary-3.2.15.tgz msgpack-2.2.0.tgz mongodb-1.14.2.tgz protobuf-3.25.2.tgz grpc-1.57.0.tgz xdebug-3.3.1.tgz"
ARG PECL_PKGS_ENABLE="igbinary msgpack mongodb protobuf grpc xdebug"

WORKDIR /pecl

COPY --from=litan1106/php-pecl:latest /pecl/ /pecl/

RUN apk add --no-cache --virtual .build-ext-deps ${PHPIZE_DEPS} \
    openssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt-dev gmp-dev \
    # linux-headers are required for building grpc extension
    libstdc++ zlib-dev linux-headers \
    && docker-php-source extract \
    && docker-php-ext-configure zip --with-zip \
    # Fix the sockets with $CFLAGS
    && CFLAGS="$CFLAGS -D_GNU_SOURCE" docker-php-ext-install -j$(nproc) mysqli pdo pdo_mysql \
    bcmath exif opcache pcntl soap xsl zip gmp sockets > /dev/null \
    && pecl install ${PECL_PKGS} > /dev/null \
    && docker-php-ext-enable ${PECL_PKGS_ENABLE} \
    && pecl bundle redis-5.3.7.tgz && cd redis \
    && phpize && ./configure --enable-redis-igbinary --enable-redis-msgpack \
    && make && make install \
    && docker-php-ext-enable redis \
    && docker-php-source delete \
    && apk del -f .build-ext-deps \
    # Setup php.ini
    && mv ${PHP_INI_DIR}/php.ini-production ${PHP_INI_DIR}/php.ini \
    && sed -i "s/memory_limit = .*/memory_limit = 1028M/" ${PHP_INI_DIR}/php.ini \
    && echo "xdebug.mode=off" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.start_with_request=yes" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini

# ==================================================================================================================
#                                                  --- BASE PHP ---
# ==================================================================================================================
FROM php:${VARIANT}

ARG PHP_MEMORY_LIMIT="4096M"
ARG OPCACHE_ENABLE=1
ARG OPCACHE_VALIDATE_TIMESTAMPS=0

ENV PHP_MEMORY_LIMIT ${PHP_MEMORY_LIMIT}
ENV OPCACHE_ENABLE ${OPCACHE_ENABLE}
ENV OPCACHE_VALIDATE_TIMESTAMPS ${OPCACHE_VALIDATE_TIMESTAMPS}
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN apk add --no-cache curl bash nano gettext \
    unzip zip libzip libxml2 libxslt \
    # Fix Iconv
    && apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.13/community/ gnu-libiconv=1.15-r3

# ----------------------------------------------------- Node.js ----------------------------------------------------
ENV NODE_VERSION 16.20.0

RUN addgroup -g 1000 node \
    && adduser -u 1000 -G node -s /bin/sh -D node \
    && apk add --no-cache \
        libstdc++ \
    && apk add --no-cache --virtual .build-deps \
        curl \
    && ARCH= && alpineArch="$(apk --print-arch)" \
      && case "${alpineArch##*-}" in \
        x86_64) \
          ARCH='x64' \
          CHECKSUM="b3f80fe7d0f1af6fe25ffedc7237ca519965d08fc800eab29cf45cd5b90cdb26" \
          ;; \
        *) ;; \
      esac \
  && if [ -n "${CHECKSUM}" ]; then \
    set -eu; \
    curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"; \
    echo "$CHECKSUM  node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" | sha256sum -c - \
      && tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
      && ln -s /usr/local/bin/node /usr/local/bin/nodejs; \
  else \
    echo "Building from source" \
    # backup build
    && apk add --no-cache --virtual .build-deps-full \
        binutils-gold \
        g++ \
        gcc \
        gnupg \
        libgcc \
        linux-headers \
        make \
        python3 \
    # use pre-existing gpg directory, see https://github.com/nodejs/docker-node/pull/1895#issuecomment-1550389150
    && export GNUPGHOME="$(mktemp -d)" \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && for key in \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      141F07595B7B3FFE74309A937405533BE57C7D57 \
      74F12602B6F1C4E913FAA37AD3A89613643B6201 \
      DD792F5973C6DE52C432CBDAC77ABFA00DDBF2B7 \
      61FC681DFB92A079F1685E77973F295594EC4689 \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      890C08DB8579162FEE0DF9DB8BEAB4DFCF555EF4 \
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
      108F52B48DB57BB0CC439B2997B01419BD92F80A \
    ; do \
      gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$key" || \
      gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xf "node-v$NODE_VERSION.tar.xz" \
    && cd "node-v$NODE_VERSION" \
    && ./configure \
    && make -j$(getconf _NPROCESSORS_ONLN) V= \
    && make install \
    && apk del .build-deps-full \
    && cd .. \
    && rm -Rf "node-v$NODE_VERSION" \
    && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt; \
  fi \
  && rm -f "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" \
  && apk del .build-deps \
  # smoke tests
  && node --version \
  && npm --version

ENV YARN_VERSION 1.22.19

RUN apk add --no-cache --virtual .build-deps-yarn curl gnupg tar \
  # use pre-existing gpg directory, see https://github.com/nodejs/docker-node/pull/1895#issuecomment-1550389150
  && export GNUPGHOME="$(mktemp -d)" \
  && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
  && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && gpgconf --kill all \
  && rm -rf "$GNUPGHOME" \
  && mkdir -p /opt \
  && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg \
  && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && apk del .build-deps-yarn \
  # smoke test
  && yarn --version

# ----------------------------------------------------- Chromium -------------------------------------------------------
ENV NPM_CONFIG_PREFIX="/opt/.npm-global"
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

RUN apk add --no-cache \
    chromium \
    nss \
    freetype \
    harfbuzz \
    ttf-freefont \
    && mkdir -p /opt/.npm-global \
    && chmod -R 777 /opt/.npm-global

USER www-data

RUN npm install --global puppeteer

USER root

# Create symlink to /usr/bin and /usr/lib
RUN mkdir -p /usr/lib/node_modules/ \
    && ln -s /opt/.npm-global/lib/node_modules/puppeteer /usr/lib/node_modules/puppeteer \
    && ln -s /usr/bin/chromium-browser /usr/bin/chromium \
    && ln -s /usr/bin/chromium-browser /usr/bin/google-chrome \
    # Create symlink to /usr/bin for node and yarn
    && ln -s /usr/local/bin/node /usr/bin/node \
    && ln -s /usr/local/bin/yarn /usr/bin/yarn

# -------------------------------------------------- Composer ---------------------------------------------------
COPY --from=composer /usr/bin/composer /usr/bin/composer
# ------------------------------------------------- RoadRunner --------------------------------------------------
COPY --from=roadrunner /usr/bin/rr /usr/bin/rr

# ----------------------------------------------- PHP-Extensions ------------------------------------------------
COPY --from=php-ext ${PHP_INI_DIR}/php.ini ${PHP_INI_DIR}/php.ini
COPY --from=php-ext ${PHP_INI_DIR}/conf.d/ ${PHP_INI_DIR}/conf.d/
COPY --from=php-ext /usr/local/lib/php/extensions/ /usr/local/lib/php/extensions/

# ------------------------------------------------ Supervisord --------------------------------------------------
ARG SUPERVISOR_USERNAME="supervisor"
ARG SUPERVISOR_PASSWORD="supervisor"

ENV SUPERVISOR_USERNAME ${SUPERVISOR_USERNAME}
ENV SUPERVISOR_PASSWORD ${SUPERVISOR_PASSWORD}

COPY --from=supervisord /usr/bin/supervisord /usr/bin/supervisord

ARG WORK_DIR="/var/www/app"

WORKDIR ${WORK_DIR}