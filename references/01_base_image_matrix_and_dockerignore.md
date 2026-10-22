# 基础镜像选型矩阵与 .dockerignore 资产排查 (Base Images & Dockerignore)

## 1. 基础镜像选型矩阵

选型核心原则：**严禁在生产运行时镜像中包含包管理器、源码构建工具或调试 Shell（极致安全与最小攻击面）**。

| 技术栈 | 推荐构建阶段 (Builder) | 推荐生产运行时 (Runtime) | 核心考量与避坑红线 |
| :--- | :--- | :--- | :--- |
| **Go 静态二进制** | `golang:1.22-alpine` 或 `golang:1.22-bookworm` | `scratch` 或 `gcr.io/distroless/static:nonroot` | 静态编译 `CGO_ENABLED=0`，零攻击面，体积 10~25MB。必须拷贝 CA 根证书与时区数据。 |
| **Python (FastAPI / 爬虫)** | `python:3.11-slim-bookworm` | `python:3.11-slim-bookworm` | 🚨 **坚决慎用 Alpine**：musl libc 缺乏预编译 wheels，编译极慢且存在内存分配器性能劣化；统一采用 Debian `slim`。 |
| **Node.js** | `node:20-bookworm-slim` | `node:20-bookworm-slim` | 同样慎用 Alpine 规避原生 C++ 扩展 (如 sharp/grpc) 编译坑。仅保留生产依赖。 |
| **PHP (Hyperf / Swoole)** | 官方 Composer 环境 | `hyperf/hyperf:8.1-alpine-v3.18-swoole` | 预装 Swoole 扩展与 OPCache 调优，锁定底层 Alpine 版本。 |

---

## 2. 生产级 `.dockerignore` 严格资产排查模板

每个项目根目录**必须**配置 `.dockerignore`，杜绝无关缓存与凭据泄露进镜像：

```text
# 版本控制与 IDE
.git
.gitignore
.idea
.vscode
*.swp

# 本地环境变量与凭据 (绝命红线)
.env*
*.pem
*.key
*.token
credentials.json

# 依赖库与构建缓存
__pycache__/
*.pyc
venv/
.venv/
node_modules/
dist/
target/
bin/

# 临时数据与日志
*.log
tmp/
scratch/
.coverage
htmlcov/
```
