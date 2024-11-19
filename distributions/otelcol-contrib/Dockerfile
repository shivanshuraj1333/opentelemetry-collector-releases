FROM alpine:3.17.0 AS jmx
ARG JMX_METRIC_GATHERER_RELEASE
RUN apk --no-cache add wget unzip && \
    JMX_METRICS_JAR=opentelemetry-jmx-metrics.jar && \
    URL=https://github.com/open-telemetry/opentelemetry-java-contrib/releases/download/${JMX_METRIC_GATHERER_RELEASE}/${JMX_METRICS_JAR} && \
    wget -O /tmp/${JMX_METRICS_JAR} "$URL" && \
    unzip -d /tmp /tmp/${JMX_METRICS_JAR}

FROM alpine:3.17.0 AS jre
RUN apk --no-cache add openjdk17-jre && \
    mkdir -p /jre && \
    cp -r /usr/lib/jvm/java-17-openjdk /jre

FROM alpine:3.19 as certs
RUN apk --no-cache add ca-certificates

FROM alpine:3.19
ARG USER_UID=10001
RUN adduser -D -u ${USER_UID} appuser
USER appuser

COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

COPY --from=jre /jre/java-17-openjdk /jre

COPY --from=jmx /tmp /tmp

COPY --chmod=755 otelcol-contrib /otelcol-contrib
COPY config.yaml /etc/otelcol-contrib/config.yaml

ENV PATH="/jre/bin:${PATH}"

ENTRYPOINT ["/otelcol-contrib"]
CMD ["--config", "/etc/otelcol-contrib/config.yaml"]

EXPOSE 4317 55678 55679
