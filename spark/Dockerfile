# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="3.12-slim-bookworm"
ARG JAVA_VERSION="17-noble"
ARG SPARK_VERSION="3.5.2-scala2.12-java17-ubuntu"

FROM eclipse-temurin:${JAVA_VERSION} as temurinjdk
FROM apache/spark:${SPARK_VERSION} as spark
# ==================================================================================================================
#                                                  --- Spark Python ---
# ==================================================================================================================
FROM python:${VARIANT}

ARG SPARK_UID=1850

LABEL description="Secured and optimized image with Python uv, Spark, and Temurin JDK."

# 1. Create a non-root user
RUN set -eux; \
    groupadd --system --gid=${SPARK_UID} spark && \
    useradd --system --uid=${SPARK_UID} --gid=spark spark

# 2. Install minimal runtime dependencies
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends gnupg2 wget bash tini libc6 libpam-modules krb5-user libnss3 procps net-tools gosu libnss-wrapper; \
    apt-get clean; \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su; \
    rm -rf /var/lib/apt/lists/*

# 3. Copy Java and Spark from official images
ENV JAVA_HOME="/opt/java/openjdk"
ENV SPARK_HOME="/opt/spark"
COPY --from=temurinjdk ${JAVA_HOME} ${JAVA_HOME}
COPY --from=spark ${SPARK_HOME} ${SPARK_HOME}
COPY --from=spark /opt/decom.sh /opt/decom.sh
COPY --from=spark /opt/entrypoint.sh /opt/entrypoint.sh

# 4. Install uv by copying binaries
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
ENV UV_CACHE_DIR=/opt/uv-cache
ENV UV_LINK_MODE=copy
ENV VENV_PATH=/opt/venv
ENV PATH="${VENV_PATH}/bin:${SPARK_HOME}/bin:${SPARK_HOME}/sbin:${JAVA_HOME}/bin:/bin:${PATH}"
ENV PYSPARK_PYTHON=${VENV_PATH}/bin/python

# 5. Create directories and set permissions
RUN set -eux; \
    mkdir -p ${UV_CACHE_DIR} ${VENV_PATH} ${SPARK_HOME}/work-dir ${SPARK_HOME}/logs /opt/spark-apps; \
    chown -R spark:spark ${UV_CACHE_DIR} ${VENV_PATH} ${SPARK_HOME}/work-dir ${SPARK_HOME}/logs /opt/spark-apps; \
    chmod +x /bin/uv /bin/uvx; \
    chmod a+x /opt/decom.sh;

# 6. Switch to non-root user and set working directory
USER spark
WORKDIR ${SPARK_HOME}/work-dir
EXPOSE 8080 7077 15002 4040
ENTRYPOINT [ "/opt/entrypoint.sh" ]
# Get Started with Spark Shell
# docker run -it custom-image-tag /opt/spark/bin/spark-shell