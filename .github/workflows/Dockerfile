FROM alpine:latest AS base
FROM base AS builder

ARG TARGETPLATFORM

COPY . /tmp/artifacts
WORKDIR /tmp/output
RUN ARTIFACT_ARCH=""; \
    if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        ARTIFACT_ARCH="x86_64"; \
    elif [ "$TARGETPLATFORM" = "linux/arm/v6" ]; then \
        ARTIFACT_ARCH="armhf"; \
    elif [ "$TARGETPLATFORM" = "linux/arm/v7" ]; then \
        ARTIFACT_ARCH="armv7hf"; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        ARTIFACT_ARCH="aarch64"; \
    elif [ "$TARGETPLATFORM" = "linux/riscv64" ]; then \
        ARTIFACT_ARCH="riscv64"; \
    else \
        echo "Unsupported architecture: $TARGETPLATFORM"; \
        exit 1; \
    fi; \
    cp /tmp/artifacts/easytier-linux-${ARTIFACT_ARCH}/* /tmp/output;

FROM base

RUN apk add --no-cache tzdata tini
WORKDIR /app
COPY --from=builder --chmod=755 /tmp/output/* /usr/local/bin

# users can use "-e TZ=xxx" to adjust it
ENV TZ Asia/Shanghai

# tcp
EXPOSE 11010/tcp 11010/udp 11011/tcp 11011/udp 11012/tcp 11211/tcp 22020/tcp 22020/udp

RUN cat > /usr/local/bin/easytier.sh <<'EOF'
#!/bin/bash
# Core
if [ "$CORE_RUN" = "norun" ]; then
    echo "easytier-core disabled."
elif [ -n "$CORE_ARGS" ]; then
    echo "Starting easytier-core with CORE_ARGS..."
    eval "easytier-core $CORE_ARGS"
else
    easytier-core
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
EOF

RUN chmod +x /usr/local/bin/easytier.sh

ENTRYPOINT ["/sbin/tini", "--", "easytier.sh"]
