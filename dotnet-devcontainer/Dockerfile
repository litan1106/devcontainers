# -------------------------------------------- Build Time Arguments ------------------------------------------------
ARG VARIANT=5.0

# ==================================================================================================================
#                                                  --- .NET ---
# ==================================================================================================================
FROM mcr.microsoft.com/dotnet/sdk:${VARIANT}

# ----------------------------------------------------- Chromium -------------------------------------------------------
ENV NPM_CONFIG_PREFIX=/opt/.npm-global
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    # Nodejs and Yarn
    && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends nodejs yarn \
    # Chrome for Puppeteer
    && apt-get -y install --no-install-recommends \
    chromium \
    fonts-freefont-ttf \
    && mkdir -p /opt/.npm-global \
    && chmod -R 777 /opt/.npm-global \
    # Util Libs
    && apt-get -y install --no-install-recommends unzip zip libzip4 libxml2 libxslt1.1 \
    libssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt1-dev \
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

# ------------------------------------------------ VsCode Setup --------------------------------------------------

# Options
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="false"
ARG ENABLE_NONROOT_DOCKER="true"
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Script locations
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh"
ARG DOCKER_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/docker-debian.sh"
ARG AZURE_CLI_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/azcli-debian.sh"
ARG COPY_KUBE_CONFIG_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/containers/kubernetes-helm/.devcontainer/copy-kube-config.sh"

ENV DOCKER_BUILDKIT=1
ENV DOTNET_CLI_HOME="/dotnet"

# Download scripts to /tmp/library-scripts
RUN mkdir /tmp/library-scripts \
    && curl -sSL $COMMON_SCRIPT_SOURCE -o /tmp/library-scripts/common-debian.sh \
    && curl -sSL $DOCKER_SCRIPT_SOURCE -o /tmp/library-scripts/docker-debian.sh \
    && curl -sSL $AZURE_CLI_SCRIPT_SOURCE -o /tmp/library-scripts/azcli-debian.sh \
    && curl -sSL $COPY_KUBE_CONFIG_SCRIPT_SOURCE -o /usr/local/share/copy-kube-config.sh

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "/var/run/docker-host.sock" "/var/run/docker.sock" "${USERNAME}" \
    && /bin/bash /tmp/library-scripts/azcli-debian.sh \
    # For VisualStudioCodeCredential
    && apt-get -y install --no-install-recommends libsecret-1-dev \
    # DotNet Home
    && mkdir -p ${DOTNET_CLI_HOME} \
    && chown ${USERNAME}:root ${DOTNET_CLI_HOME} \
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

ARG WORK_DIR="/workspace"

WORKDIR ${WORK_DIR}
