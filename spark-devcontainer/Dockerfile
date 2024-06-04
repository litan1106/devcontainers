# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="3.5.1-scala2.12-java17-ubuntu"

# ==================================================================================================================
#                                                  --- Spark ---
# ==================================================================================================================
FROM spark:${VARIANT}

USER root

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    #
    # Utils and Libs
    && apt-get -y install --no-install-recommends software-properties-common gettext-base nano \
    unzip zip libzip4 libxml2 libxslt1.1 libssl-dev libgcrypt-dev libzip-dev libxml2-dev libxslt1-dev \
    #
    # Python 3.11
    && add-apt-repository ppa:deadsnakes/ppa \
    && apt-get update \
    && apt-get -y install --no-install-recommends python3.11 python3-pip \
    #
    # Chrome for Puppeteer
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1 \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# ------------------------------------------------ VsCode Setup --------------------------------------------------
# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"
# [Option] Enable non-root Docker access in container
ARG ENABLE_NONROOT_DOCKER="true"
# [Option] Use the OSS Moby CLI instead of the licensed Docker CLI
ARG USE_MOBY="true"
# [Option] Select CLI version
ARG CLI_VERSION="latest"

# Enable new "BUILDKIT" mode for Docker CLI
ENV DOCKER_BUILDKIT=1

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Script locations
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/common-debian.sh"
ARG DOCKER_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/docker-debian.sh"
ARG AZURE_CLI_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/azcli-debian.sh"
ARG COPY_KUBE_CONFIG_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/containers/kubernetes-helm/.devcontainer/copy-kube-config.sh"

# Download scripts to /tmp/library-scripts
RUN mkdir /tmp/library-scripts \
    && curl -sSL $COMMON_SCRIPT_SOURCE -o /tmp/library-scripts/common-debian.sh \
    && curl -sSL $DOCKER_SCRIPT_SOURCE -o /tmp/library-scripts/docker-debian.sh \
    && curl -sSL $AZURE_CLI_SCRIPT_SOURCE -o /tmp/library-scripts/azcli-debian.sh \
    && curl -sSL $COPY_KUBE_CONFIG_SCRIPT_SOURCE -o /usr/local/share/copy-kube-config.sh

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "/var/run/docker-host.sock" "/var/run/docker.sock" "${USERNAME}" "${USE_MOBY}" "${CLI_VERSION}" \
    # Install the Azure CLI
    && /bin/bash /tmp/library-scripts/azcli-debian.sh \
    # Install the kubectl
    && curl -sSL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    # Install the Helm
    && curl -s https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash - \
    && chown ${USERNAME}:root /usr/local/share/copy-kube-config.sh \
    && echo "source /usr/local/share/copy-kube-config.sh" | tee -a /root/.bashrc /root/.zshrc /home/${USERNAME}/.bashrc >> /home/${USERNAME}/.zshrc \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to 
# the Docker socket if "overrideCommand": false is set in devcontainer.json. 
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>