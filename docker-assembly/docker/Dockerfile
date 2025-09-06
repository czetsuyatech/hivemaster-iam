# Build hivemaster-iam

FROM eclipse-temurin:21.0.8_9-jre as builder

# Environment Variables

ENV KEYCLOAK_VERSION 24.0.2
ENV LAUNCH_JBOSS_IN_BACKGROUND 1
ENV PROXY_ADDRESS_FORWARDING false
ENV KEYCLOAK_HOME /opt/jboss/keycloak
ENV LANG en_US.UTF-8

# Bash Arguments

ARG KEYCLOAK_DIST_BASE=https://github.com/keycloak/keycloak/releases/download/$KEYCLOAK_VERSION/
ARG KEYCLOAK_DIST_FILE=keycloak-$KEYCLOAK_VERSION.tar.gz

# Base

USER root

# Install the necessary packages

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-transport-https ca-certificates curl jsvc unzip \
# dos2unix
        dos2unix \
# git
        git \
# procps : for 'free' command
        procps \
# iptools-ping : for 'ping' command
#        iptools-ping \
# iproute2 : for 'ip' command
        iproute2 \
# java.lang.UnsatisfiedLinkError: /usr/local/openjdk-11/lib/libfontmanager.so: libfreetype.so.6: cannot open shared object file: No such file or directory
# java.lang.NoClassDefFoundError: Could not initialize class sun.awt.X11FontManager
# https://github.com/docker-library/openjdk/pull/235#issuecomment-424466077
        fontconfig libfreetype6 \
    && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

# Configurations

RUN mkdir /opt/jboss

ADD src/main/resources/tools /opt/jboss/tools
RUN chmod +x /opt/jboss/tools/*
RUN dos2unix /opt/jboss/tools/*

ADD downloads /opt/jboss/keycloak_install_stage/downloads

ADD target/keycloak_providers /opt/jboss/keycloak_install_stage/keycloak_providers

ADD src/main/resources/realms /opt/jboss/keycloak_install_stage/realms

# Build Keycloak

RUN /opt/jboss/tools/build-keycloak.sh

# Start Keycloak

FROM eclipse-temurin:21.0.8_9-jre

RUN groupadd -r keycloak -g 1001 \
    && useradd -u 1001 -r -g keycloak -m -d /opt/keycloak -s /sbin/nologin -c "Keycloak User" keycloak

COPY --from=builder /opt/jboss/keycloak /opt/keycloak
COPY --from=builder /opt/jboss/tools /opt/keycloak/tools

RUN chown -R keycloak:keycloak /opt/keycloak \
  && chmod -R g+rwx /opt/keycloak

USER keycloak

# Ensure signals are forwarded to the JVM process correctly for graceful shutdown
ENV LAUNCH_JBOSS_IN_BACKGROUND true

# Expose Keycloak ports
EXPOSE 8080
EXPOSE 8443
EXPOSE 9990

ENTRYPOINT ["/opt/keycloak/tools/docker-entrypoint.sh"]

CMD ["-b", "0.0.0.0"]
