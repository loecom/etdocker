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

ARG ROOT_PASSWORD

RUN apt update && apt install -y --no-install-recommends tzdata tini openssh-server && ssh-keygen -A && mkdir -p /run/sshd
RUN echo "root:${ROOT_PASSWORD}" | chpasswd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
WORKDIR /app
COPY --from=builder --chmod=755 /tmp/output/* /usr/local/bin

# users can use "-e TZ=xxx" to adjust it
ENV TZ Asia/Shanghai

# tcp
EXPOSE 22 80 443 1433 3306 5244 8000 8080 8082 8384 11010 11010/udp 11011 11011/udp 11012 11211 22020 22020/udp

RUN printf '#!/bin/sh\n\
/usr/sbin/sshd -D &\n\
easytier-core &\n\
easytier-web-embed -d /data/et.db' > /usr/local/bin/runet.sh \
    && chmod +x /usr/local/bin/runet.sh

ENTRYPOINT ["/usr/bin/tini", "--", "runet.sh"]
