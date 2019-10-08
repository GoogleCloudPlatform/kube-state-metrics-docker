# Download and unpack the Kube State Merics release code from github
FROM gcr.io/google-appengine/debian9 as downloader

ENV KUBE_STATE_METRICS_VERSION 1.8.0
ENV KUBE_STATE_METRICS_URL="https://github.com/kubernetes/kube-state-metrics/archive/v$KUBE_STATE_METRICS_VERSION.tar.gz"
ENV KUBE_STATE_METRICS_SHA256 4fcf035b0be2696b714e425905adaf090729b2fafe8d6defc5bd421c07426237

RUN set -x && \
    apt-get update && \
    apt-get install -qq -y wget && \
    mkdir release && \
    wget -O release.tar.gz "$KUBE_STATE_METRICS_URL" && \
    echo "$KUBE_STATE_METRICS_SHA256 release.tar.gz" | sha256sum -c - && \
    tar xfvz release.tar.gz --strip-components=1 -C release


# Build the binary with Go
FROM golang:1.12 as builder
WORKDIR /go/src/k8s.io/kube-state-metrics

ENV PKG=k8s.io/kube-state-metrics
ENV TAG=1.8.0

ENV GCO_ENABLED=0
ENV GOOS=linux

COPY --from=downloader release /go/src/k8s.io/kube-state-metrics

RUN set -x && \
    BuildDate="$(date -u +%Y-%m-%dT%H:%M:%SZ)" && \
    go build \
      -ldflags="-s -w -X ${PKG}/version.Release=${TAG} -X ${PKG}/version.BuildDate=${BuildDate}" \
      -o kube-state-metrics


# Build the final image
FROM gcr.io/google-appengine/debian9

ENV C2D_RELEASE=1.8.0

COPY --from=downloader /release/LICENSE /kube-state-metrics-LICENSE
COPY --from=builder /go/src/k8s.io/kube-state-metrics/kube-state-metrics /

VOLUME /tmp

EXPOSE 8080
EXPOSE 8081

ENTRYPOINT ["/kube-state-metrics", "--port=8080", "--telemetry-port=8081"]
