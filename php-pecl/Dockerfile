# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.1-cli-alpine3.17"

# ==================================================================================================================
#                                                --- PHP PECL ---
# ==================================================================================================================
FROM php:${VARIANT}

ARG PECL_PKGS="redis-5.3.7 igbinary-3.2.12 msgpack-2.1.2 mongodb-1.14.2 protobuf-3.21.9 grpc-1.49.0 xdebug-3.1.6"

WORKDIR /pecl

# Download pecl packages
RUN pecl channel-update pecl.php.net \
    && pecl download ${PECL_PKGS}

# Check pecl packages
RUN tar -xzf redis-5.3.7.tgz \
    && tar -xzf igbinary-3.2.12.tgz \
    && tar -xzf msgpack-2.1.2.tgz \
    && tar -xzf mongodb-1.14.2.tgz \
    && tar -xzf protobuf-3.21.9.tgz \
    && tar -xzf grpc-1.49.0.tgz \
    && tar -xzf xdebug-3.1.6.tgz