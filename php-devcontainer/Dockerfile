# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.0-cli"
ARG COMPOSER_VERSION="2.0"
ARG ROADRUNNER_VERSION="2.7.3"

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

ARG PECL_PKGS="igbinary-3.2.6.tgz msgpack-2.1.2.tgz mongodb-1.11.1.tgz protobuf-3.19.1.tgz grpc-1.41.0.tgz xdebug-3.1.1.tgz"
ARG PECL_PKGS_ENABLE="igbinary msgpack mongodb protobuf grpc xdebug"

WORKDIR /pecl

COPY --from=litan1106/php-pecl:latest /pecl/ /pecl/

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends libssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt1-dev libgmp-dev \
    && docker-php-source extract \
    && docker-php-ext-configure zip --with-zip \
    && docker-php-ext-install -j$(nproc) mysqli pdo pdo_mysql \
    bcmath exif opcache pcntl soap xsl zip gmp sockets > /dev/null \
    && pecl install ${PECL_PKGS} > /dev/null \
    && docker-php-ext-enable ${PECL_PKGS_ENABLE} \
    && pecl bundle redis && cd redis \
    && phpize && ./configure --enable-redis-igbinary --enable-redis-msgpack \
    && make && make install \
    && docker-php-ext-enable redis \
    && docker-php-source delete \
    # Setup php.ini
    && mv ${PHP_INI_DIR}/php.ini-development ${PHP_INI_DIR}/php.ini \
    && sed -i "s/memory_limit = .*/memory_limit = 1028M/" ${PHP_INI_DIR}/php.ini \
    && echo "xdebug.mode=off" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.start_with_request=yes" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> ${PHP_INI_DIR}/conf.d/docker-php-ext-xdebug.ini \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

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

ENV NPM_CONFIG_PREFIX=/opt/.npm-global
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    # Required pkgs for Zip, SOAP and XML
    && apt-get -y install --no-install-recommends unzip zip libzip4 libxml2 libxslt1.1 \
    # Tools
    gettext-base nano redis-tools \
    # Nodejs and Yarn for Puppeteer and Websocket
    && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends nodejs yarn \
    # Chrome for Puppeteer
    && apt-get -y install --no-install-recommends chromium fonts-freefont-ttf \
    && mkdir -p /opt/.npm-global \
    && chmod -R 777 /opt/.npm-global \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

USER $USERNAME

RUN npm install --global puppeteer

USER root
# Create symlink to /usr/bin and /usr/lib
RUN mkdir -p /usr/lib/node_modules/ \
    && ln -s /opt/.npm-global/lib/node_modules/puppeteer /usr/lib/node_modules/puppeteer \
    && ln -s /usr/bin/chromium /usr/bin/google-chrome

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

# ------------------------------------------------ VsCode Setup --------------------------------------------------
# Options
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="false"
ARG ENABLE_NONROOT_DOCKER="true"
ARG USE_MOBY="false"
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Script locations
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh"
ARG DOCKER_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/containers/docker-in-docker/.devcontainer/library-scripts/docker-in-docker-debian.sh"
ARG AZURE_CLI_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/azcli-debian.sh"
ARG COPY_KUBE_CONFIG_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/containers/kubernetes-helm/.devcontainer/copy-kube-config.sh"

# Download scripts to /tmp/library-scripts
RUN mkdir /tmp/library-scripts \
    && curl -sSL $COMMON_SCRIPT_SOURCE -o /tmp/library-scripts/common-debian.sh \
    && curl -sSL $DOCKER_SCRIPT_SOURCE -o /tmp/library-scripts/docker-in-docker-debian.sh \
    && curl -sSL $AZURE_CLI_SCRIPT_SOURCE -o /tmp/library-scripts/azcli-debian.sh \
    && curl -sSL $COPY_KUBE_CONFIG_SCRIPT_SOURCE -o /usr/local/share/copy-kube-config.sh

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-in-docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "${USERNAME}" "${USE_MOBY}" \
    && /bin/bash /tmp/library-scripts/azcli-debian.sh \
    # Install kubectl
    && curl -sSL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    # Install Helm
    && curl -s https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash - \
    && chown ${USERNAME}:root /usr/local/share/copy-kube-config.sh \
    && echo "source /usr/local/share/copy-kube-config.sh" | tee -a /root/.bashrc /root/.zshrc /home/${USERNAME}/.bashrc >> /home/${USERNAME}/.zshrc \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

ARG WORK_DIR="/workspace"

WORKDIR ${WORK_DIR}