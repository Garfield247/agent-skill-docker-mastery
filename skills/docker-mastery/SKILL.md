---
name: docker-mastery
description: >-
  Docker 生产级容器化架构、多阶段瘦身构建、安全基线与微服务编排规范技能。
  基于 Docker 官方最佳实践与 CIS Docker Benchmark 安全基线。
  涵盖多阶段构建 (Multi-stage Build) 极致瘦身、基础镜像选型矩阵 (Go scratch/distroless, Python/Node slim 防 musl 坑)、
  Layer 缓存优化与变动频次排序、.dockerignore 严格资产排查、Non-root 专属用户最小权限安全红线、
  1号进程 (PID 1) 信号转发与僵死进程治理 (tini/dumb-init)、优雅停机 (STOPSIGNAL SIGTERM)、
  原生 HEALTHCHECK 探针标准、Docker Compose 网络隔离编排及 Go/Python/PHP 生产级标杆 Dockerfile。
---

# Docker 生产级容器化架构与安全工程规范 (Docker Mastery Skill)

本技能定义了构建高安全、极致瘦身、平滑信号停机的生产级 Docker 镜像与容器编排时的核心门禁与渐进式手册索引。

---

## 1. 核心铁律与绝对红线 (Core Redlines)

1. 🚨 **基础镜像选型与避免 Alpine 坑**：
   - 生产运行环境严禁残留编译器与调试工具；
   - Go 语言统一使用 `scratch` 或 `distroless`（10~25MB）；
   - **Python/Node 坚决慎用 Alpine**（musl 缺少预编译 wheel 且内存分配性能劣化），统一采用 Debian `slim-bookworm`。
2. 🚨 **变动频次排序与缓存清理闭环**：
   - 依赖文件拷贝与安装必须置于应用源码拷贝之前，充分利用构建缓存；
   - 包管理器安装与缓存清理（`rm -rf /var/lib/apt/lists/*`）必须位于**同一 `RUN` 指令内**。
3. 🚨 **Non-root 专属用户与零硬编码密钥**：
   - **坚决禁止使用 Root 运行业务容器**，必须显式创建并切换至 UID >= 10001 的非特权系统用户；
   - 严禁通过 `ENV` 或 `ARG` 固化数据库密码、API Key 等机密数据。
4. 🚨 **PID 1 进程治理与优雅停机**：
   - Python/Node.js 容器必须引入 `tini` 充当 PID 1 收割僵尸进程；
   - 容器启动指令**必须采用 Exec 数组格式**（`CMD ["node", "index.js"]`），严禁使用 Shell 格式（防止无法响应 `SIGTERM` 导致被暴力强杀）。

---

## 2. 标准开发工作流 (Standard Workflow)

1. **资产排查**：在项目根目录部署标准的 `.dockerignore` 模板；
2. **多阶段构建**：分离构建环境（含 SDK）与纯净运行时环境；
3. **用户降权**：创建非特权用户 `appuser` 并授予目标目录最小读写权限；
4. **探针与信号**：配置 `STOPSIGNAL SIGTERM`、`tini` 与原生 `HEALTHCHECK`；
5. **构建校验**：运行 `docker build` 检查最终镜像层数、体积与非 root 运行状态。

---

## 3. 渐进式专业参考手册 (Progressive Disclosure References)

根据具体涉及的业务场景，**按需查阅 `references/` 目录下的专门规约**，严禁将全量子文档一次性无脑全部展开：

| 场景模块 | 参考子文档 | 核心涵盖规范 |
| :--- | :--- | :--- |
| **基础镜像与忽略资产** | [01_base_image_matrix_and_dockerignore.md](references/01_base_image_matrix_and_dockerignore.md) | • Go/Python/Node/PHP 镜像选型对照表<br>• 生产级 `.dockerignore` 严格资产排查模板 |
| **多阶段构建与分层缓存**| [02_multistage_and_layer_caching.md](references/02_multistage_and_layer_caching.md) | • 变动频次由低到高排序架构图<br>• 同一 Layer 缓存清理军规与体积压缩 |
| **安全加固与 Non-root** | [03_security_hardening_and_nonroot.md](references/03_security_hardening_and_nonroot.md) | • CIS Docker Benchmark 安全红线清单<br>• UID 10001 非特权用户创建与权限最小化 |
| **PID 1 与探针治理** | [04_pid1_signals_and_healthcheck.md](references/04_pid1_signals_and_healthcheck.md) | • tini 僵尸进程收割与信号转发机理<br>• Exec 格式 vs Shell 格式停机陷阱<br>• 原生 HEALTHCHECK 探针编写标准 |
| **标杆模版与 Compose 编排**| [05_dockerfile_templates_and_compose.md](references/05_dockerfile_templates_and_compose.md) | • Go Scratch 15MB 标杆 Dockerfile<br>• Python FastAPI Slim 生产标杆 Dockerfile<br>• Docker Compose 后端内网隔离编排 |
