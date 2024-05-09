FROM node:20-slim AS frontend-build
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
COPY . /app
WORKDIR /app
RUN pnpm install --frozen-lockfile
RUN pnpm install -g concurrently
RUN NODE_ENV=production DISABLE_TYPECHECKING=true pnpm build-web

FROM golang:1.22.1-alpine as bob-the-builder
RUN apk add --no-cache bash
WORKDIR /app
COPY --from=frontend-build /app .
RUN mkdir /build
RUN go build -trimpath -ldflags '-X github.com/sourcegraph/sourcegraph/internal/version.version=20067_2018-09-06_11e7ddb -X github.com/sourcegraph/sourcegraph/internal/version.timestamp=1714576964' -buildmode exe -installsuffix netgo -tags 'dist netgo' -o /build/frontend "github.com/sourcegraph/sourcegraph/cmd/frontend"


FROM sourcegraph/alpine-3.14:213466_2023-04-17_5.0-bdda34a71619@sha256:6354a4ff578b685e36c8fbde81f62125ae0011b047fb2cc22d1b0de616b3c59a

ARG COMMIT_SHA="unknown"
ARG DATE="unknown"
ARG VERSION="unknown"

LABEL org.opencontainers.image.revision=${COMMIT_SHA}
LABEL org.opencontainers.image.created=${DATE}
LABEL org.opencontainers.image.version=${VERSION}
LABEL com.sourcegraph.github.url=https://github.com/sourcegraph/sourcegraph/commit/${COMMIT_SHA}

ENV CONFIGURATION_MODE=server PGDATABASE=sg PGHOST=pgsql PGPORT=5432 PGSSLMODE=disable PGUSER=sg CODEINTEL_PGDATABASE=sg CODEINTEL_PGHOST=codeintel-db CODEINTEL_PGPORT=5432 CODEINTEL_PGSSLMODE=disable CODEINTEL_PGUSER=sg PUBLIC_REPO_REDIRECTS=true
COPY --from=bob-the-builder /build/frontend /usr/local/bin/frontend
USER sourcegraph
CMD ["serve"]
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/frontend"]
