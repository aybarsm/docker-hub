# syntax=docker/dockerfile:1

FROM alpine:latest
LABEL maintainer="aybarsm" tag_build="$BUILD_DATE"

RUN set -eux \
    && apk add --no-cache mini_httpd \
    && rm -rf /tmp/* /var/tmp/*

WORKDIR /var/www/localhost/htdocs

EXPOSE 80
EXPOSE 443

ENTRYPOINT ["mini_httpd", "-D"]