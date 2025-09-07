# syntax=docker/dockerfile:1

FROM semaphoreui/semaphore:latest AS build

WORKDIR /tmp

USER 0

COPY requirements.txt /tmp/requirements.txt

RUN set -eux; \
    apk update; \
    apk add --no-cache gcc musl-dev cairo-dev pkgconfig python3-dev; \
    SEMAPHORE_VENV="$VIRTUAL_ENV"; \
    cd "$SEMAPHORE_VENV"; \
    . ./bin/activate; \
    pip3 install --upgrade pip setuptools wheel; \
    pip3 install --upgrade -r "/tmp/requirements.txt"; \
    cp -r "$(python3 -c 'import site; print(site.getsitepackages()[0])')" /tmp/site_packages; \
    deactivate

FROM semaphoreui/semaphore:latest
LABEL maintainer="aybarsm" tag_build="20250907"

USER 0

COPY --from=build /tmp/site_packages /tmp/site_packages
COPY requirements.yml /tmp/requirements.yml

RUN set -eux; \
    export SEMAPHORE_VENV="$VIRTUAL_ENV"; \
    cd "$SEMAPHORE_VENV"; \
    . ./bin/activate; \
    export PATH_SITE_PACKAGES="$(python3 -c 'import site; print(site.getsitepackages()[0])')"; \
    rm -rf "$PATH_SITE_PACKAGES"; \
    cp -r /tmp/site_packages "$PATH_SITE_PACKAGES"; \
    ansible-galaxy install -r /tmp/requirements.yml --force-with-deps; \
    deactivate; \
    chown -R semaphore:0 /opt/semaphore; \
    rm -rf /tmp/* /var/tmp/*

USER 1001