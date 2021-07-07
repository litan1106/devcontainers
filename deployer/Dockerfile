# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.0-cli-alpine"
ARG COMPOSER_VERSION="2.0"

# ------------------------------------------------- Composer Image -------------------------------------------------
FROM composer:${COMPOSER_VERSION} as composer

# ==================================================================================================================
#                                                  --- BASE PHP ---
# ==================================================================================================================
FROM php:${VARIANT}

ENV PATH "${PATH}:/root/.composer/vendor/bin"
ENV COMPOSER_ALLOW_SUPERUSER 1

RUN apk add --no-cache curl bash nano git \
    unzip zip libzip

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN composer global require "laravel/envoy=^2.0"

ARG WORK_DIR="/var/www/app"

WORKDIR ${WORK_DIR}

ENTRYPOINT [ "envoy" ]