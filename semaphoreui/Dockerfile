# syntax=docker/dockerfile:1

FROM semaphoreui/semaphore:latest AS build

WORKDIR /tmp

USER 0

COPY requirements.txt /tmp/requirements.txt
COPY requirements.yml /tmp/requirements.yml

RUN set -eux \
    && apk update \
    && apk add --no-cache gcc musl-dev cairo-dev pkgconfig python3-dev \
    && source "$VIRTUAL_ENV/bin/activate" \
    && export PATH_SITE_PACKAGES="$(python3 -c 'import site; print(site.getsitepackages()[0])')" \
    && echo "$PATH_SITE_PACKAGES" | tee /tmp/path_site_packages >/dev/null \
    && pip3 install --upgrade -r "/tmp/requirements.txt" \
    && mkdir -p "/usr/share/ansible/roles" \
    && mkdir -p "/usr/share/ansible/collections" \
    && ansible-galaxy collection install -r /tmp/requirements.yml --force-with-deps --collections-path "/usr/share/ansible/collections" \
    && ansible-galaxy role install -r /tmp/requirements.yml --force-with-deps --roles-path "/usr/share/ansible/roles" \
    && cp -r "$PATH_SITE_PACKAGES" /tmp/site_packages

FROM semaphoreui/semaphore:latest
LABEL maintainer="aybarsm" tag_build="20250907"

USER 0

COPY --from=build /tmp/site_packages /tmp/site_packages
COPY --from=build /tmp/path_site_packages /tmp/path_site_packages
COPY --from=build /usr/share/ansible /usr/share/ansible

RUN set -eux \
    && export PATH_SITE_PACKAGES="$(cat /tmp/path_site_packages)" \
    && rm -rf "$PATH_SITE_PACKAGES" \
    && cp -r /tmp/site_packages "$PATH_SITE_PACKAGES" \
    && chown -R semaphore:0 /opt/semaphore \
    && chown -R semaphore:0 /usr/share/ansible \
    && rm -rf /tmp/* /var/tmp/*

ENV VIRTUAL_ENV_PACKAGES="$(cat /tmp/path_site_packages)"

USER 1001