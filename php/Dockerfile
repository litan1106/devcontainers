# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.0-fpm-alpine"
ARG COMPOSER_VERSION="2.0"
ARG ROADRUNNER_VERSION="2.0.4"

# ------------------------------------------------- Composer Image -------------------------------------------------
FROM composer:${COMPOSER_VERSION} as composer

# ------------------------------------------------- RoadRunner Image -----------------------------------------------
FROM spiralscout/roadrunner:${ROADRUNNER_VERSION} as roadrunner

# ==================================================================================================================
#                                                  --- BASE PHP ---
# ==================================================================================================================
FROM php:${VARIANT}

ARG WORK_DIR="/var/www/app"

WORKDIR ${WORK_DIR}

ARG PECL_PHP_DEP_VERSIONS="igbinary-3.2.2 msgpack-2.1.2 mongodb-1.9.1 xdebug-3.0.4 swoole-4.6.6"
ARG PECL_PHP_DEPS="igbinary msgpack mongodb xdebug swoole"

ARG PHP_MEMORY_LIMIT="4096M"
ARG OPCACHE_ENABLE=1
ARG OPCACHE_VALIDATE_TIMESTAMPS=0
ARG PHP_INI_TEMPLATE_DIR="/usr/local/phpini/templates/"

ENV PHP_MEMORY_LIMIT ${PHP_MEMORY_LIMIT}
ENV OPCACHE_ENABLE ${OPCACHE_ENABLE}
ENV OPCACHE_VALIDATE_TIMESTAMPS ${OPCACHE_VALIDATE_TIMESTAMPS}
ENV PHP_INI_TEMPLATE_DIR ${PHP_INI_TEMPLATE_DIR}

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN apk add --no-cache curl bash nano gettext \
    unzip zip libzip libxml2 libxslt \
    # Fix Iconv
    gnu-libiconv

# ==================================================================================================================
#                                                  --- SUPERVISOR ---
# ==================================================================================================================
ARG SUPERVISOR_USERNAME="supervisor"
ARG SUPERVISOR_PASSWORD="supervisor"

ENV SUPERVISOR_USERNAME ${SUPERVISOR_USERNAME}
ENV SUPERVISOR_PASSWORD ${SUPERVISOR_PASSWORD}

# ----------------------------------------------------- Python -----------------------------------------------------
RUN apk add --no-cache python3 \
    && apk add --no-cache --virtual .build-deps build-base python3-dev py3-pip \
    && python3 -m pip install --upgrade pip setuptools \
    && python3 -m pip install supervisor \
    && apk del .build-deps

# ----------------------------------------------------- Node.js ----------------------------------------------------
ENV NODE_VERSION 14.16.1

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
          CHECKSUM="39fbbd09668472f1105bdb5a7c20675bd91649543d2bc25b6edc1293835661f1" \
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
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && for key in \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      74F12602B6F1C4E913FAA37AD3A89613643B6201 \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
      108F52B48DB57BB0CC439B2997B01419BD92F80A \
      B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
      gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
      gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
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

ENV YARN_VERSION 1.22.5

RUN apk add --no-cache --virtual .build-deps-yarn curl gnupg tar \
  && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
  && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && mkdir -p /opt \
  && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg \
  && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && apk del .build-deps-yarn \
  # smoke test
  && yarn --version

# ----------------------------------------------------- Chromium -------------------------------------------------------
ENV NPM_CONFIG_PREFIX=/opt/.npm-global
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

# ------------------------------------------------ Extensions --------------------------------------------------
RUN apk add --no-cache --virtual .build-ext-deps ${PHPIZE_DEPS} \
    openssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt-dev gmp-dev \
    && docker-php-source extract \
    && docker-php-ext-configure zip --with-zip \
    && docker-php-ext-install -j$(nproc) mysqli pdo pdo_mysql \
    bcmath exif opcache pcntl soap xsl zip gmp sockets > /dev/null \
    && pecl install ${PECL_PHP_DEP_VERSIONS} > /dev/null \
    && docker-php-ext-enable ${PECL_PHP_DEPS} \
    && pecl bundle redis && cd redis \
    && phpize && ./configure --enable-redis-igbinary --enable-redis-msgpack \
    && make && make install \
    && docker-php-ext-enable redis \
    && docker-php-source delete \
    && apk del -f .build-ext-deps \
    # Setup php.ini
    && mv ${PHP_INI_DIR}/php.ini-production ${PHP_INI_DIR}/php.ini \
    && rm -rf /var/www/* /usr/local/etc/php-fpm.d/* \
    && sed -i "s/memory_limit = .*/memory_limit = 1028M/" ${PHP_INI_DIR}/php.ini \
    && echo "xdebug.mode=off" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.start_with_request=yes" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini

COPY --from=composer /usr/bin/composer /usr/bin/composer
COPY --from=roadrunner /usr/bin/rr /usr/bin/rr