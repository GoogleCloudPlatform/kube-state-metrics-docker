# Download and unpack the Kube State Merics release code from github
FROM gcr.io/google-appengine/debian9 as downloader

ENV KUBE_STATE_METRICS_VERSION 2.2.0

RUN set -x && \
    apt-get update && \
    apt-get install -qq -y wget git && \
    git clone https://github.com/kubernetes/kube-state-metrics.git --branch v${KUBE_STATE_METRICS_VERSION} release

# Build the binary with Go
FROM golang:1.16 as builder
WORKDIR /go/src/k8s.io/kube-state-metrics

ENV PKG=k8s.io/kube-state-metrics
ENV TAG=2.2.0

ENV GCO_ENABLED=0
ENV GOOS=linux

COPY --from=downloader release /go/src/k8s.io/kube-state-metrics

RUN set -x && \
    BuildDate="$(date -u +%Y-%m-%dT%H:%M:%SZ)" && \
    go build \
      -ldflags="-s -w -X ${PKG}/version.Release=${TAG} -X ${PKG}/version.BuildDate=${BuildDate}" \
      -o kube-state-metrics

# Build the final image
FROM marketplace.gcr.io/google/debian9

ENV C2D_RELEASE=2.2.0

COPY --from=downloader /release/LICENSE /kube-state-metrics-LICENSE
COPY --from=builder /go/src/k8s.io/kube-state-metrics/kube-state-metrics /

VOLUME /tmp
USER nobody

EXPOSE 8080
EXPOSE 8081

ENTRYPOINT ["/kube-state-metrics", "--port=8080", "--telemetry-port=8081"]
