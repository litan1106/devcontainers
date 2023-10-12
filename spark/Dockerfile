# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG PYTHON_MAJOR_VERSION="3.11"

# ==================================================================================================================
#                                                  --- Spark and Python ---
# ==================================================================================================================
FROM python:${PYTHON_MAJOR_VERSION}-bullseye

ENV PYTHONUNBUFFERED 1
ENV JAVA_HOME /opt/java/openjdk
ENV SPARK_HOME /opt/spark

# Build multi stage from official spark image
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu $JAVA_HOME $JAVA_HOME
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu $SPARK_HOME $SPARK_HOME
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu /opt/decom.sh /opt/decom.sh
COPY --from=apache/spark:3.5.0-scala2.12-java11-ubuntu /opt/entrypoint.sh /opt/entrypoint.sh

WORKDIR ${SPARK_HOME}/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]
