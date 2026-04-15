FROM --platform=$BUILDPLATFORM golang:1.25-alpine3.21 AS builder

RUN apk add --no-cache \
    ca-certificates \
    git \
    tzdata

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

COPY . .

ARG TARGETOS
ARG TARGETARCH
ARG TARGETVARIANT

RUN GOARM="${TARGETVARIANT#v}" \
    GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS="${TARGETOS:-linux}" \
    GOARCH="${TARGETARCH:-amd64}" \
    go build -a -trimpath \
    -ldflags="-s -w -extldflags '-static' -X github.com/containrrr/watchtower/internal/meta.Version=$(git describe --tags --always)" \
    -o /watchtower .

FROM alpine:3.21
LABEL "com.centurylinklabs.watchtower"="true"

RUN apk add --no-cache \
    ca-certificates \
    tzdata

EXPOSE 8080

COPY --from=builder /watchtower /watchtower

HEALTHCHECK CMD [ "/watchtower", "--health-check"]

ENTRYPOINT ["/watchtower"]
