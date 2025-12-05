FROM linuxserver/code-server:latest

USER root

# Install system deps and SUSHI (FSH compiler) via npm
RUN apt-get update \
  && apt-get install -y curl git npm ca-certificates --no-install-recommends \
  && npm install -g fsh-sushi \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Copy extensions list and install extensions (lines starting with # are ignored)
COPY extensions.txt /tmp/extensions.txt
RUN set -eux; \
  while IFS= read -r ext || [ -n "$ext" ]; do \
    case "$ext" in \#*|"") continue ;; esac; \
    code-server --install-extension "$ext" || true; \
  done < /tmp/extensions.txt

WORKDIR /config/workspace/

