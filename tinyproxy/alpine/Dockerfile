# syntax=docker/dockerfile:1

FROM alpine:latest
LABEL maintainer="aybarsm" tag_build="$BUILD_DATE"

RUN set -eux \
    && apk add --no-cache tinyproxy curl

EXPOSE 8888

ENTRYPOINT ["/usr/bin/tinyproxy", "-d"]
    
