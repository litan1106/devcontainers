# -------------------------------------------- Build Time Arguments ------------------------------------------------
ARG VARIANT=8.0-cli

# ==================================================================================================================
#                                                  --- PHP ---
# ==================================================================================================================
FROM php:${VARIANT}

# Options
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="false"
ARG ENABLE_NONROOT_DOCKER="true"
ARG SOURCE_SOCKET=/var/run/docker-host.sock
ARG TARGET_SOCKET=/var/run/docker.sock
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Script locations
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh"
ARG DOCKER_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/docker-debian.sh"
ARG AZURE_CLI_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/azcli-debian.sh"
ARG COPY_KUBE_CONFIG_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/containers/kubernetes-helm/.devcontainer/copy-kube-config.sh"

# Download scripts to /tmp/library-scripts
RUN mkdir /tmp/library-scripts \
    && curl -sSL $COMMON_SCRIPT_SOURCE -o /tmp/library-scripts/common-debian.sh \
    && curl -sSL $DOCKER_SCRIPT_SOURCE -o /tmp/library-scripts/docker-debian.sh \
    && curl -sSL $AZURE_CLI_SCRIPT_SOURCE -o /tmp/library-scripts/azcli-debian.sh \
    && curl -sSL $COPY_KUBE_CONFIG_SCRIPT_SOURCE -o /usr/local/share/copy-kube-config.sh

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "${SOURCE_SOCKET}" "${TARGET_SOCKET}" "${USERNAME}" \
    && /bin/bash /tmp/library-scripts/azcli-debian.sh \
    # Clean up
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

# Install kubectl
RUN curl -sSL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
    && chmod +x /usr/local/bin/kubectl

# Install Helm
RUN curl -s https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash -

# Script copies localhost's ~/.kube/config file into the container and swaps out
# localhost for host.docker.internal on bash/zsh start to keep them in sync.
RUN chown ${USERNAME}:root /usr/local/share/copy-kube-config.sh \
    && echo "source /usr/local/share/copy-kube-config.sh" | tee -a /root/.bashrc /root/.zshrc /home/${USERNAME}/.bashrc >> /home/${USERNAME}/.zshrc

# PHP Extensions
ARG PECL_PHP_DEP_VERSIONS="igbinary-3.1.6 msgpack-2.1.2 mongodb-1.9.0"
ARG PECL_PHP_DEPS="igbinary msgpack mongodb"

ARG PHP_MEMORY_LIMIT="4096M"
ARG OPCACHE_ENABLE=1
ARG OPCACHE_VALIDATE_TIMESTAMPS=0

ENV PHP_MEMORY_LIMIT ${PHP_MEMORY_LIMIT}
ENV OPCACHE_ENABLE ${OPCACHE_ENABLE}
ENV OPCACHE_VALIDATE_TIMESTAMPS ${OPCACHE_VALIDATE_TIMESTAMPS}

# Composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# Supervisor
ARG SUPERVISOR_USERNAME="supervisor"
ARG SUPERVISOR_PASSWORD="supervisor"

ENV SUPERVISOR_USERNAME ${SUPERVISOR_USERNAME}
ENV SUPERVISOR_PASSWORD ${SUPERVISOR_PASSWORD}

# Puppeteer
ENV NPM_CONFIG_PREFIX=/opt/.npm-global
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

ENV ROADRUNNER_VERSION="2.0.4"

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    #
    # Python
    && apt-get -y install --no-install-recommends python3 python3-dev python3-pip python3-venv nginx redis-tools \
    && python3 -m pip install --upgrade pip setuptools supervisor \
    #
    # PHP Extensions
    && apt-get -y install --no-install-recommends unzip zip libzip4 libxml2 libxslt1.1 \
    libssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt1-dev libgmp-dev \
    && docker-php-source extract \
    && docker-php-ext-configure zip --with-zip \
    && docker-php-ext-install -j$(nproc) mysqli pdo pdo_mysql \
    bcmath exif opcache pcntl soap xsl zip gmp > /dev/null \
    && pecl install ${PECL_PHP_DEP_VERSIONS} > /dev/null \
    && docker-php-ext-enable ${PECL_PHP_DEPS} \
    && pecl bundle redis && cd redis \
    && phpize && ./configure --enable-redis-igbinary --enable-redis-msgpack && make && make install \
    && docker-php-ext-enable redis \
    && pecl install xdebug > /dev/null \
    && docker-php-ext-enable xdebug \
    && docker-php-source delete \
    #
    # Setup php.ini
    && mv ${PHP_INI_DIR}/php.ini-development ${PHP_INI_DIR}/php.ini \
    && rm -rf /var/www/* \
    && rm -rf /etc/nginx/conf.d/* \
    #
    # Nodejs and Yarn
    && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends nodejs yarn \
    #
    # Chrome for Puppeteer
    && apt-get -y install --no-install-recommends \
    chromium \
    fonts-freefont-ttf \
    && mkdir -p /opt/.npm-global \
    && chmod -R 777 /opt/.npm-global \
    #
    # Roadrunner
    && curl -sL https://github.com/spiral/roadrunner-binary/releases/download/v${ROADRUNNER_VERSION}/roadrunner-${ROADRUNNER_VERSION}-linux-amd64.tar.gz -o /tmp/roadrunner.tar.gz \
    && tar -xf /tmp/roadrunner.tar.gz -C /usr/bin --strip-components=1 \
    && rm -rf /tmp/roadrunner.tar.gz \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Install npm puppeteer
USER $USERNAME

RUN npm install --global puppeteer

USER root
# Create symlink to /usr/bin and /usr/lib
RUN mkdir -p /usr/lib/node_modules/ \
    && ln -s /opt/.npm-global/lib/node_modules/puppeteer /usr/lib/node_modules/puppeteer \
    && ln -s /usr/bin/chromium /usr/bin/google-chrome

COPY xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

EXPOSE 8000

# LARAVEL
WORKDIR /workspace

ENV APP_URL "http://localhost"
ENV APP_ENV "local"
ENV APP_DEBUG true
ENV LOG_CHANNEL "errorlog"
ENV DB_HOST "mysql"
ENV REDIS_HOST "redis"
ENV MONGODB_HOST "mongo"
ENV DUSK_DRIVER_URL "localhost:9515"
# Back to user
USER $USERNAME
