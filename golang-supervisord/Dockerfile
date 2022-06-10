# syntax=docker/dockerfile:1.4
# -------------------------------------------- Build Time Arguments ------------------------------------------------
ARG ALPINE_VERSION="3.15"
ARG GO_VERSION="1.17"

# ==================================================================================================================
#                                                  --- SUPERVISORD ---
# ==================================================================================================================
FROM golang:${GO_VERSION}-alpine AS supervisord-builder

ENV GOPROXY=direct 
ENV GOSUMDB=off 
ENV CGO_ENABLED=0

RUN apk add --no-cache --virtual .build-deps git gcc rust \
    && mkdir -p /root/gocode \
    && export GOPATH=/root/gocode \
    && go mod init supervisord-builder \
    && go get github.com/ochinchina/supervisord \
    && go build -a -ldflags "-linkmode external -extldflags -static" -o /usr/local/bin/supervisord github.com/ochinchina/supervisord \
    && rm -rf /root/gocode \
    && apk del -f .build-deps

# Copy supervisord from builder
FROM alpine:${ALPINE_VERSION}

COPY --from=supervisord-builder /usr/local/bin/supervisord /usr/bin/

ENTRYPOINT ["/usr/bin/supervisord"]