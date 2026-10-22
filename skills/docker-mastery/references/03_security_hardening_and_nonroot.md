# 容器安全加固与 Non-root 最小权限红线 (Security Hardening)

## 1. 生产环境安全五大绝命红线

1. 🚨 **坚决禁止使用 Root 运行业务容器 (Non-root Rule)**：
   - 必须显式创建无登录权限的专属系统用户（UID/GID >= 10001）；
   - 使用 `USER appuser:appuser` 切换，杜绝容器逃逸危害宿主机内核。
2. 🚨 **严禁在镜像层中硬编码敏感凭据 (Zero Hardcoded Secrets)**：
   - 严禁通过 `ENV` 或 `ARG` 固化 API 密钥、数据库密码或 Token（`docker history` 直接泄露）；
   - 敏感信息必须通过运行时环境变量或挂载 Volume 注入。
3. 🚨 **基础镜像标签显式锁定 (No Mutable :latest)**：
   - 坚决杜绝使用 `:latest`。必须锁定主次版本号（如 `python:3.11.9-slim-bookworm`）。
4. 🚨 **生产文件系统权限最小化**：
   - 仅授予运行时用户对数据目录或日志目录的读写权限，代码目录保持只读权限。

---

## 2. 标杆非特权用户创建指令

```dockerfile
# Debian / Ubuntu 环境
RUN groupadd -g 10001 appuser && \
    useradd -u 10001 -g appuser -s /bin/false -M -d /app appuser

# Alpine 环境
RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10001 -S appuser -G appgroup -s /bin/false -H
```
