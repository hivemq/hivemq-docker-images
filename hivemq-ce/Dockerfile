FROM openjdk:11-jre-slim

ARG HIVEMQ_CE_VERSION=2019.1
ARG HIVEMQ_GID=10000
ARG HIVEMQ_UID=10000

# Additional JVM options, may be overwritten by user
ENV JAVA_OPTS "-XX:+UseNUMA"

# We use tini for proper signal propagation and a clean init
ENV TINI_VERSION v0.18.0

RUN set -x \
    && apt-get update && apt-get install -y --no-install-recommends curl gnupg gnupg-agent dirmngr \
    && curl -fSL "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini" -o /usr/local/bin/tini \
    && curl -fSL "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini.asc" -o /usr/local/bin/tini.asc \
    && export GNUPGHOME="$(mktemp -d)" \
    && echo "disable-ipv6" >> "$GNUPGHOME"/dirmngr.conf \
    && gpg --no-tty --keyserver ha.pool.sks-keyservers.net --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 \
    && gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
    && rm -rf "$GNUPGHOME" /usr/local/bin/tini.asc \
    && chmod +x /usr/local/bin/tini \
    && apt-get remove -y gnupg gnupg-agent dirmngr && rm -rf /var/lib/apt/lists/*

COPY config.xml /opt/config.xml

# HiveMQ setup
RUN curl -L https://github.com/hivemq/hivemq-community-edition/releases/download/${HIVEMQ_CE_VERSION}/hivemq-ce-${HIVEMQ_CE_VERSION}.zip -o /opt/hivemq-${HIVEMQ_CE_VERSION}.zip \
    && unzip /opt/hivemq-${HIVEMQ_CE_VERSION}.zip  -d /opt/\
    && rm -f /opt/hivemq-${HIVEMQ_CE_VERSION}.zip \
    && ln -s /opt/hivemq-ce-${HIVEMQ_CE_VERSION} /opt/hivemq \
    && mv /opt/config.xml /opt/hivemq-ce-${HIVEMQ_CE_VERSION}/conf/config.xml \
    && groupadd --gid ${HIVEMQ_GID} hivemq \
    && useradd -g hivemq -d /opt/hivemq -s /bin/bash --uid ${HIVEMQ_UID} hivemq \
    && chown -R hivemq:hivemq /opt/hivemq-ce-${HIVEMQ_CE_VERSION} \
    && chmod -R 777 /opt \
    && chmod +x /opt/hivemq/bin/run.sh

# Substitute eval for exec and replace OOM flag if necessary (for older releases). This is necessary for proper signal propagation
RUN sed -i -e 's|eval \\"java\\" "$HOME_OPT" "$JAVA_OPTS" -jar "$JAR_PATH"|exec "java" $HOME_OPT $JAVA_OPTS -jar "$JAR_PATH"|' /opt/hivemq/bin/run.sh && \
    sed -i -e "s|-XX:OnOutOfMemoryError='sleep 5; kill -9 %p'|-XX:+CrashOnOutOfMemoryError|" /opt/hivemq/bin/run.sh

# Make broker data persistent throughout stop/start cycles
VOLUME /opt/hivemq/data

# Persist log data
VOLUME /opt/hivemq/log

EXPOSE 1883

WORKDIR /opt/hivemq
USER ${HIVEMQ_UID}

ENTRYPOINT ["/usr/local/bin/tini", "-g", "--"]
CMD ["/opt/hivemq/bin/run.sh"]
