FROM ubuntu:latest AS builder

ARG TARGETPLATFORM

COPY . /tmp/artifacts
WORKDIR /tmp/output
RUN ARTIFACT_ARCH=""; \
    if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        ARTIFACT_ARCH="x86_64"; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        ARTIFACT_ARCH="aarch64"; \
    else \
        echo "Unsupported architecture: $TARGETPLATFORM"; \
        exit 1; \
    fi; \
    cp /tmp/artifacts/easytier-linux-${ARTIFACT_ARCH}/* /tmp/output;

FROM ubuntu:latest

RUN apt update && apt install -y --no-install-recommends tzdata tini openssh-server && ssh-keygen -A && mkdir -p /run/sshd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
WORKDIR /app
COPY --from=builder --chmod=755 /tmp/output/* /usr/local/bin

# users can use "-e TZ=xxx" to adjust it
ENV TZ Asia/Shanghai

# tcp
EXPOSE 22 80 443 1433 3306 5244 8000 8080 8082 8384 11010 11010/udp 11011 11011/udp 11012 11211 22020 22020/udp

RUN cat > /usr/local/bin/easytier.sh <<'EOF'
#!/bin/bash
set -e

if [ -n "$ROOT_PASSWORD" ]; then
    echo "root:$ROOT_PASSWORD" | chpasswd
    echo "Root password set from environment."
fi

# Core
if [ "$CORE_RUN" = "norun" ]; then
    echo "easytier-core disabled."
elif [ -n "$CORE_ARGS" ]; then
    echo "Starting easytier-core with CORE_ARGS..."
    eval "easytier-core $CORE_ARGS" &
else
    easytier-core &
fi

# Web
if [ "$WEB_RUN" = "norun" ]; then
    echo "easytier-web-embed disabled."
elif [ -n "$WEB_ARGS" ]; then
    echo "Starting easytier-web-embed with WEB_ARGS..."
    eval "easytier-web-embed $WEB_ARGS" &
else
    easytier-web-embed &
fi

exec /usr/sbin/sshd -D
EOF

RUN chmod +x /usr/local/bin/easytier.sh

ENTRYPOINT ["/usr/bin/tini", "--", "easytier.sh"]
