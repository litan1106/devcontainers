# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.0-cli-alpine3.15"

# ==================================================================================================================
#                                                --- PHP PECL ---
# ==================================================================================================================
FROM php:${VARIANT}

ARG PECL_PKGS="redis-5.3.7 igbinary-3.2.7 msgpack-2.1.2 mongodb-1.12.1 protobuf-3.19.4 grpc-1.44.0 xdebug-3.1.3"

WORKDIR /pecl

# Download pecl packages
RUN pecl channel-update pecl.php.net \
    && pecl download ${PECL_PKGS}

# Check pecl packages
RUN tar -xzf redis-5.3.7.tgz \
    && tar -xzf igbinary-3.2.7.tgz \
    && tar -xzf msgpack-2.1.2.tgz \
    && tar -xzf mongodb-1.12.1.tgz \
    && tar -xzf protobuf-3.19.4.tgz \
    && tar -xzf grpc-1.44.0.tgz \
    && tar -xzf xdebug-3.1.3.tgz