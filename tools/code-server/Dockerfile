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

# Add a container init script for linuxserver base that will install extensions
# into the mounted /config/extensions directory on every container start.
RUN mkdir -p /etc/cont-init.d && \
    cat > /etc/cont-init.d/50-install-extensions.sh <<'EOF'
#!/bin/sh
set -eu

# Install listed extensions into the persistent extensions dir on every start.
EXT_DIR=/config/extensions
EXT_LIST=/tmp/extensions.txt

echo "Installing extensions from $EXT_LIST into $EXT_DIR"
if [ -f "$EXT_LIST" ]; then
  while IFS= read -r ext || [ -n "$ext" ]; do
    # strip CRLF and surrounding whitespace
    ext="$(echo "$ext" | tr -d '\r' | sed -e 's/^\s*//' -e 's/\s*$//')"
    case "$ext" in
      \#*|"") continue ;;
    esac
    echo "Installing extension: $ext"
    # run as the code-server runtime user
    su -s /bin/sh abc -c "/app/code-server/bin/code-server --install-extension $ext --extensions-dir $EXT_DIR" || true
  done < "$EXT_LIST"
else
  echo "$EXT_LIST not found; no extensions to install"
fi

EOF
RUN chmod +x /etc/cont-init.d/50-install-extensions.sh || true && \
  # Ensure script has Unix line endings (strip CRLF if present)
  sed -i 's/\r$//' /etc/cont-init.d/50-install-extensions.sh || true

