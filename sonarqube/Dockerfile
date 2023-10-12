# syntax=docker/dockerfile:1.4
# ----------------------------------------------- Build Time Arguments ---------------------------------------------
ARG VARIANT="8.9-community"

# ==================================================================================================================
#                                                --- Build PDF Plugins---
# ==================================================================================================================
FROM maven:3-openjdk-11 as custom-plugins

WORKDIR /opt

ARG PDF_PLUGIN_URL="https://github.com/sleroy/sonar-pdf-report/archive/refs/tags/1.5.0.tar.gz"
ENV PDF_PLUGIN_URL ${PDF_PLUGIN_URL}

RUN mkdir -p sonar-pdf-report-src
RUN curl -L ${PDF_PLUGIN_URL} -o sonar-pdf-report.tar.gz
RUN tar -xf sonar-pdf-report.tar.gz -C sonar-pdf-report-src --strip-components=1
RUN rm sonar-pdf-report.tar.gz
RUN cd sonar-pdf-report-src && mvn clean install

# ==================================================================================================================
#                                              --- SonarQube with Custom Plugins---
# ==================================================================================================================
FROM sonarqube:${VARIANT}

COPY --from=custom-plugins /opt/sonar-pdf-report-src/target/sonar-pdf-report*.jar /opt/sonarqube/extensions/plugins/