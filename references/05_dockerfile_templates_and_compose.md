# 核心技术标杆 Dockerfile 与 Compose 编排规范 (Templates & Compose)

## 1. Go 语言 (Scratch 超轻量 15MB 标杆 Dockerfile)

```dockerfile
FROM golang:1.22-bookworm AS builder
ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -trimpath -ldflags="-s -w -buildid=" -o /build/app-service .

FROM alpine:3.19 AS certs
RUN apk --no-cache add ca-certificates tzdata

FROM scratch AS runtime
WORKDIR /app
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=certs /usr/share/zoneinfo /usr/share/zoneinfo
ENV TZ=Asia/Shanghai
COPY --from=builder /build/app-service /app/app-service
COPY --from=builder /build/etc /app/etc
EXPOSE 8888
STOPSIGNAL SIGTERM
ENTRYPOINT ["/app/app-service", "-f", "etc/app.yaml"]
```

---

## 2. Python (FastAPI 极简非 root 生产级模版)

```dockerfile
FROM python:3.11-slim-bookworm AS builder
WORKDIR /install
COPY requirements.txt .
RUN pip install --prefix=/install/deps -r requirements.txt

FROM python:3.11-slim-bookworm AS runtime
ENV PATH="/app/deps/bin:$PATH" \
    PYTHONPATH="/app/deps/lib/python3.11/site-packages:$PYTHONPATH"
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates tzdata tini curl \
    && rm -rf /var/lib/apt/lists/*
RUN groupadd -g 10001 appuser && useradd -u 10001 -g appuser -s /bin/false -M appuser
COPY --from=builder --chown=appuser:appuser /install/deps /app/deps
COPY --chown=appuser:appuser . /app/
USER appuser:appuser
EXPOSE 8000
STOPSIGNAL SIGTERM
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://127.0.0.1:8000/healthz || exit 1
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 3. Docker Compose 生产编排网络隔离规范

```yaml
version: '3.8'

networks:
  frontend:
    driver: bridge
  backend:
    internal: true # 🚨 后端内网隔离，禁止向宿主机外部暴露端口

services:
  mysql:
    image: mysql:8.0.36
    restart: unless-stopped
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3

  api:
    build: .
    restart: unless-stopped
    networks:
      - frontend
      - backend
    ports:
      - "8000:8000"
    depends_on:
      mysql:
        condition: service_healthy # 🚨 等待数据库探针就绪再启动服务
```
