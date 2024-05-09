FROM golang:1.22.1-alpine as bob-the-builder
RUN apk add --no-cache bash
WORKDIR /app
COPY . .
RUN mkdir /build
RUN go build -trimpath -ldflags '-X github.com/sourcegraph/sourcegraph/internal/version.version=0.0.0-DEVELOPMENT -X github.com/sourcegraph/sourcegraph/internal/version.timestamp=1714576964' -buildmode exe -installsuffix netgo -tags 'dist netgo' -o /build/searcher ./cmd/searcher


FROM sourcegraph/alpine-3.14:213466_2023-04-17_5.0-bdda34a71619@sha256:6354a4ff578b685e36c8fbde81f62125ae0011b047fb2cc22d1b0de616b3c59a

RUN apk --no-cache add pcre sqlite-libs libev

# The comby/comby image is a small binary-only distribution. See the bin and src directories
# here: https://github.com/comby-tools/comby/tree/master/dockerfiles/alpine
# hadolint ignore=DL3022
COPY --from=comby/comby:alpine-3.14-1.8.1@sha256:a5e80d6bad6af008478679809dc8327ebde7aeff7b23505b11b20e36aa62a0b2 /usr/local/bin/comby /usr/local/bin/comby

ARG COMMIT_SHA="unknown"
ARG DATE="unknown"
ARG VERSION="unknown"

LABEL org.opencontainers.image.revision=${COMMIT_SHA}
LABEL org.opencontainers.image.created=${DATE}
LABEL org.opencontainers.image.version=${VERSION}
LABEL com.sourcegraph.github.url=https://github.com/sourcegraph/sourcegraph/commit/${COMMIT_SHA}

# Use SEARCHER_CACHE_DIR to set the cache dir at runtime for searcher. Setting CACHE_DIR will also
# apply to other services and is deprecated.
ENV CACHE_DIR=/mnt/cache/searcher
RUN mkdir -p ${CACHE_DIR} && chown -R sourcegraph:sourcegraph ${CACHE_DIR}
COPY --from=bob-the-builder /build/searcher /usr/local/bin/searcher
USER sourcegraph
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/searcher"]
