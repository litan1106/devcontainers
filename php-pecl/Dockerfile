# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.0-cli-alpine3.13"

# ==================================================================================================================
#                                                --- PHP PECL ---
# ==================================================================================================================
FROM php:${VARIANT}

ARG PECL_PKGS="igbinary-3.2.6 msgpack-2.1.2 mongodb-1.10.0 protobuf-3.17.3 grpc-1.39.0 xdebug-3.0.4"

WORKDIR /pecl

# Download pecl packages
RUN pecl channel-update pecl.php.net \
    && pecl download ${PECL_PKGS}

# Check pecl packages
RUN tar -xzf igbinary-3.2.6.tgz \
    && tar -xzf msgpack-2.1.2.tgz \
    && tar -xzf mongodb-1.10.0.tgz \
    && tar -xzf protobuf-3.17.3.tgz \
    && tar -xzf grpc-1.39.0.tgz \
    && tar -xzf xdebug-3.0.4.tgz