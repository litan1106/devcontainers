# syntax=docker/dockerfile:1.4
# --- Build Arguments for Version Overrides ---
ARG PYTHON_VERSION_ARG="3.12"
ARG CODENAME_ARG="slim-bookworm"
ARG SPARK_VERSION_ARG="3.5.2"
ARG HADOOP_VERSION_ARG="3"
ARG JAVA_VERSION_ARG="17"

# --- STAGE 1: Spark and Java Builder ---
FROM python:${PYTHON_VERSION_ARG}-${CODENAME_ARG} AS sparkbuilder

ARG SPARK_VERSION_ARG
ARG HADOOP_VERSION_ARG
ARG JAVA_VERSION_ARG

# 1. Install dependencies
RUN set -eux; \
    DEBIAN_FRONTEND=noninteractive apt-get update -y && apt-get install -y --no-install-recommends \
    curl wget ca-certificates gnupg software-properties-common apt-transport-https && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 2. Install Eclipse Temurin JDK
ENV JAVA_HOME_BUILDER="/usr/lib/jvm/temurin-${JAVA_VERSION_ARG}-jdk"
RUN set -eux; \
    echo "Setting up Adoptium APT repository and installing Temurin JDK ${JAVA_VERSION_ARG}..."; \
    wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor > /usr/share/keyrings/adoptium.gpg; \
    echo "deb [signed-by=/usr/share/keyrings/adoptium.gpg] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME=/{print$2}' /etc/os-release) main" > /etc/apt/sources.list.d/adoptium.list; \
    DEBIAN_FRONTEND=noninteractive apt-get update -y && apt-get install -y --no-install-recommends temurin-${JAVA_VERSION_ARG}-jdk; \
    echo "Temurin JDK ${JAVA_VERSION_ARG} installed in ${JAVA_HOME_BUILDER}"; \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Download and extract Apache Spark
ENV SPARK_TGZ_URL_BUILDER=https://archive.apache.org/dist/spark/spark-${SPARK_VERSION_ARG}/spark-${SPARK_VERSION_ARG}-bin-hadoop${HADOOP_VERSION_ARG}.tgz
ENV SPARK_HOME_BUILDER=/opt/spark-dist
RUN set -eux; \
    mkdir -p ${SPARK_HOME_BUILDER}; \
    wget -nv -O spark.tgz "$SPARK_TGZ_URL_BUILDER"; \
    tar -xzf spark.tgz -C ${SPARK_HOME_BUILDER} --strip-components=1; \
    rm spark.tgz*

# --- STAGE 2: Final Image ---
FROM python:${PYTHON_VERSION_ARG}-${CODENAME_ARG}

ARG JAVA_VERSION_ARG
ARG SPARK_UID=1850

LABEL description="Secure, cache-optimized image with uv, Python, Spark, and parametrized Temurin JDK (via APT)."

# 1. Create a non-root user
RUN set -eux; \
    groupadd --system --gid=${SPARK_UID} spark && \
    useradd --system --uid=${SPARK_UID} --gid=spark spark

# 2. Install minimal runtime dependencies
RUN set -eux; \
    DEBIAN_FRONTEND=noninteractive apt-get update -y && apt-get install -y --no-install-recommends tini libnss-wrapper bash && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Copy Java and Spark from builder stage
ENV JAVA_HOME="/usr/lib/jvm/temurin-${JAVA_VERSION_ARG}-jdk"
ENV SPARK_HOME=/opt/spark
COPY --chown=spark:spark --from=sparkbuilder ${JAVA_HOME_BUILDER} ${JAVA_HOME}
COPY --chown=spark:spark --from=sparkbuilder ${SPARK_HOME_BUILDER} ${SPARK_HOME}

# 4. Install uv by copying binaries
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
RUN set -eux; chmod +x /bin/uv /bin/uvx

ENV UV_CACHE_DIR=/opt/uv-cache
ENV UV_LINK_MODE=copy
ENV VENV_PATH=/opt/venv
ENV PATH="${VENV_PATH}/bin:${SPARK_HOME}/bin:${SPARK_HOME}/sbin:${JAVA_HOME}/bin:/bin:${PATH}"
ENV PYSPARK_PYTHON=${VENV_PATH}/bin/python

# 5. Create directories and set permissions
RUN set -eux; \
    mkdir -p ${UV_CACHE_DIR} ${VENV_PATH} ${SPARK_HOME}/work-dir ${SPARK_HOME}/logs /opt/spark-apps && \
    chown -R spark:spark ${UV_CACHE_DIR} ${VENV_PATH} ${SPARK_HOME}/work-dir ${SPARK_HOME}/logs /opt/spark-apps 

# 6. Switch to non-root user and set working directory
USER spark
WORKDIR ${SPARK_HOME}/work-dir
EXPOSE 8080 7077 15002 4040
ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "bash" ]
