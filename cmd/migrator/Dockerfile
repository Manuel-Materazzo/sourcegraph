FROM golang:1.22.1-alpine as bob-the-builder
RUN apk add --no-cache bash
WORKDIR /app
COPY . .
RUN mkdir /build
RUN go build -trimpath -ldflags '-X github.com/sourcegraph/sourcegraph/internal/version.version=0.0.0-DEVELOPMENT -X github.com/sourcegraph/sourcegraph/internal/version.timestamp=1714576964' -buildmode exe -installsuffix netgo -tags 'dist netgo' -o /build/migrator ./cmd/migrator

FROM sourcegraph/alpine-3.14:213466_2023-04-17_5.0-bdda34a71619@sha256:6354a4ff578b685e36c8fbde81f62125ae0011b047fb2cc22d1b0de616b3c59a

ARG COMMIT_SHA="unknown"
ARG DATE="unknown"
ARG VERSION="unknown"

LABEL org.opencontainers.image.revision=${COMMIT_SHA}
LABEL org.opencontainers.image.created=${DATE}
LABEL org.opencontainers.image.version=${VERSION}
LABEL com.sourcegraph.github.url=https://github.com/sourcegraph/sourcegraph/commit/${COMMIT_SHA}

RUN apk update && apk add --no-cache \
    tini

COPY --from=bob-the-builder /build/migrator /usr/local/bin/migrator
USER sourcegraph
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/migrator"]
#COPY schema-descriptions /schema-descriptions
