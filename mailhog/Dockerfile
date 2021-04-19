# -------------------------------------------- Build Time Arguments ------------------------------------------------
ARG ALPINE_VERSION="3.13"
ARG GO_VERSION="1.16"

# ==================================================================================================================
#                                                  --- MAILHOG ---
# ==================================================================================================================
FROM golang:${GO_VERSION}-alpine AS mailhog-builder

RUN apk add --no-cache --virtual .build-deps git \
    && mkdir -p /root/gocode \
    && export GOPATH=/root/gocode \
    && go get github.com/mailhog/MailHog \
    && mv /root/gocode/bin/MailHog /usr/local/bin \
    && rm -rf /root/gocode \
    && apk del -f .build-deps

# Copy MailHog from builder
FROM alpine:${ALPINE_VERSION}

ARG MH_USERNAME="mailhog"
ARG MH_PASSWORD="mailhog"
ENV MH_USERNAME=${MH_USERNAME}
ENV MH_PASSWORD=${MH_PASSWORD}

WORKDIR /home/mailhog

RUN adduser -D -u 1000 mailhog

USER mailhog

COPY --from=mailhog-builder /usr/local/bin/MailHog /usr/bin/

ENTRYPOINT echo $MH_USERNAME:$(MailHog bcrypt $MH_PASSWORD) > auth_file && MailHog -auth-file=auth_file

# SMTP and HTTP
EXPOSE 1025 8025
