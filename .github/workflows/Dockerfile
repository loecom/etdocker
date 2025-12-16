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
#!/bin/sh
set -e

# 设置 root 密码（如果 ROOT_PASSWORD 环境变量存在）
if [ -n "$ROOT_PASSWORD" ]; then
    echo "root:$ROOT_PASSWORD" | chpasswd
    echo "Root password set from environment."
fi

# 可选：创建普通用户（如果 USERNAME 和 PASSWORD 提供了）
if [ -n "$USERNAME" ] && [ -n "$PASSWORD" ]; then
    # 如果用户不存在则创建
    if ! id "$USERNAME" >/dev/null 2>&1; then
        adduser --disabled-password --gecos '' "$USERNAME"
        echo "$USERNAME:$PASSWORD" | chpasswd
        # 可选：加入 sudo 组（需安装 sudo）
        # adduser "$USERNAME" sudo
    else
        echo "$USERNAME:$PASSWORD" | chpasswd
    fi
    echo "User '$USERNAME' configured."
fi

# 启动服务
/usr/sbin/sshd -D &
easytier-core &
easytier-web-embed -p ws -d /data/et.db &
EOF

RUN chmod +x /usr/local/bin/easytier.sh

ENTRYPOINT ["/usr/bin/tini", "--", "easytier.sh"]
