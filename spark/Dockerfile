# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG PYTHON_MAJOR_VERSION="3.11"

# ==================================================================================================================
#                                                  --- Spark and Python ---
# ==================================================================================================================
FROM python:${PYTHON_MAJOR_VERSION}-bullseye

ARG SPARK_VERSION="3.5.0"
ARG SCALA_VERSION="2.12"
ARG PYSPARK_VERSION="3.4.1"
ARG POETRY_VERSION="1.6.1"
ARG SPARK_UID=185

ENV CI 1
ENV PYTHONUNBUFFERED 1
ENV POETRY_HOME /opt/poetry
ENV POETRY_NO_INTERACTION 1
ENV POETRY_VIRTUALENVS_IN_PROJECT 1
ENV POETRY_VIRTUALENVS_CREATE 1
ENV POETRY_CACHE_DIR /tmp/poetry_cache
ENV JAVA_HOME /opt/java/openjdk
ENV SPARK_HOME /opt/spark
ENV PATH "${POETRY_HOME}/bin:${PATH}"

WORKDIR /opt/spark/work-dir

# Build multi stage from official spark image
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu $JAVA_HOME $JAVA_HOME
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu /opt/spark /opt/spark
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu /opt/decom.sh /opt/decom.sh
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu /opt/entrypoint.sh /opt/entrypoint.sh

USER root

RUN groupadd --system --gid=${SPARK_UID} spark && \
    useradd --system --uid=${SPARK_UID} --gid=spark spark \
    && mkdir -p $POETRY_HOME \
    && mkdir -p $POETRY_CACHE_DIR \
    && chown -R spark:spark $POETRY_HOME \
    && chown -R spark:spark $POETRY_CACHE_DIR \
    && chown -R spark:spark /opt/spark

USER spark

# Install poetry
RUN python -m venv $POETRY_HOME \
    && ${POETRY_HOME}/bin/pip install "poetry==$POETRY_VERSION"

ENTRYPOINT [ "/opt/entrypoint.sh" ]
