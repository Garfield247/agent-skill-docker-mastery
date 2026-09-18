# Docker 生产级容器化架构与安全工程规范技能 (`docker-mastery`)

生产级容器化打包、多阶段构建瘦身、镜像安全基线、进程治理与微服务编排技能，对齐 Docker 官方最佳实践与 CIS Docker Benchmark。

## 🎯 核心规范一览

- **基础镜像矩阵**：Go 极简 distroless/scratch、Python/Node 选用 Debian slim 规避 Alpine musl 性能与编译陷阱；
- **多阶段构建与 Layer 缓存**：按变动频次严格排序，编译环境与运行时彻底物理隔离；
- **容器安全底线**：强制 Non-root 专属用户运行 (UID 10001)，坚决禁止镜像硬编码敏感凭据；
- **进程生命周期与优雅停机**：引入 tini/dumb-init 治理 PID 1 僵死进程与信号转发，强制 Exec 格式与 STOPSIGNAL；
- **HEALTHCHECK 与编排**：原生健康探针配置标准，Docker Compose 网络内网隔离与依赖探针就绪编排。

## 🚀 安装与接入

```bash
# 通过 agy 或中央技能库安装
agy skill install Garfield247/agent-skill-docker-mastery
```
