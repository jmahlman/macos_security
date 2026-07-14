# syntax=docker/dockerfile:1
FROM alpine:3.21@sha256:48b0309ca019d89d40f670aa1bc06e426dc0931948452e8491e3d65087abc07d

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

RUN --mount=type=secret,id=corp_ca,target=/tmp/ca.pem,required=false \
    if [ -f /tmp/ca.pem ]; then cat /tmp/ca.pem >> /etc/ssl/certs/ca-certificates.crt; fi && \
    apk update && apk add --no-cache \
  python3 \
  ruby \
  git \
  ruby-dev \
  build-base \
  libjpeg-turbo \
  libpng \
  freetype \
  libxml2 \
  libxslt \
  yaml \
  zsh

RUN --mount=type=secret,id=corp_ca,target=/tmp/ca.pem,required=false \
    if [ -f /tmp/ca.pem ]; then cat /tmp/ca.pem >> /etc/ssl/certs/ca-certificates.crt; fi && \
    apk add --no-cache --virtual .build-deps \
    musl-dev \
    linux-headers \
    g++ \
    gcc \
    zlib-dev \
    make \
    python3-dev \
    jpeg-dev \
    freetype-dev \
    libpng-dev \
    openblas-dev \
    libxml2-dev \
    libxslt-dev \
    yaml-dev \
    rust \
    cargo

ARG BUILD_SHA=unknown
ARG IMAGE_REPO=unknown
ENV MSCP_BUILD_SHA=${BUILD_SHA}
ENV MSCP_IMAGE_REPO=${IMAGE_REPO}

WORKDIR /mscp

# Install Python dependencies before copying source (layer cache is not invalidated by code changes)
COPY requirements.txt ./
RUN --mount=type=secret,id=corp_ca,target=/tmp/ca.pem,required=false \
    if [ -f /tmp/ca.pem ]; then cat /tmp/ca.pem >> /etc/ssl/certs/ca-certificates.crt; fi && \
    uv venv /opt/venv && \
    UV_NATIVE_TLS=1 uv pip install --no-cache --python /opt/venv/bin/python -r requirements.txt
ENV PATH="/opt/venv/bin:$PATH"

# Install Ruby dependencies before copying source
COPY Gemfile ./
RUN --mount=type=secret,id=corp_ca,target=/tmp/ca.pem,required=false \
    if [ -f /tmp/ca.pem ]; then cat /tmp/ca.pem >> /etc/ssl/certs/ca-certificates.crt; fi && \
    gem install bundler && bundle install

COPY . .

# Install the mscp package itself so `mscp` is on PATH (in addition to ./mscp.py)
RUN --mount=type=secret,id=corp_ca,target=/tmp/ca.pem,required=false \
    if [ -f /tmp/ca.pem ]; then cat /tmp/ca.pem >> /etc/ssl/certs/ca-certificates.crt; fi && \
    UV_NATIVE_TLS=1 uv pip install --no-cache --python /opt/venv/bin/python .

RUN apk del .build-deps

RUN adduser -D -u 1001 -s /bin/zsh mscp && chown -R mscp:mscp /mscp && \
    chmod +x /mscp/.github/container/entrypoint.sh

# Enable argcomplete for the mscp CLI in the mscp user's zsh sessions
RUN printf '%s\n' \
    'autoload -U compinit && compinit' \
    'autoload -U bashcompinit && bashcompinit' \
    'eval "$(register-python-argcomplete mscp)"' \
    'eval "$(register-python-argcomplete mscp.py)"' \
    "PROMPT='%~ %# '" \
    > /home/mscp/.zshrc && chown mscp:mscp /home/mscp/.zshrc

USER mscp

ENTRYPOINT ["/mscp/.github/container/entrypoint.sh"]
CMD ["zsh"]
