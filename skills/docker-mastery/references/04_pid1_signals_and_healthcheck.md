# PID 1 进程治理、信号平滑透传与 HEALTHCHECK 规范 (Process & Healthcheck)

## 1. PID 1 僵尸进程与信号转发机理

在 Linux 容器中，容器启动命令即为 **1 号进程 (PID 1)**：
- **信号转发**：Docker 发送 `SIGTERM` 时，PID 1 必须捕获并正确透传给子进程；
- **孤儿进程收割 (Reaping)**：子进程派生出的孤儿进程必须由 PID 1 回收，否则会产生僵尸进程耗尽系统 PID 资源；
- **治理方案**：Go 语言作为单二进制可直接运行并捕获系统信号；Python/Node.js/Shell 容器**必须引入 `tini` 或 `dumb-init` 充当 PID 1**。

---

## 2. 优雅停机三要素与 Exec 格式

```dockerfile
# 1. 显式指定停机信号
STOPSIGNAL SIGTERM

# 2. 引入 tini 充当 PID 1
RUN apt-get update && apt-get install -y --no-install-recommends tini && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["/usr/bin/tini", "--"]

# 3. 🚨 必须使用 Exec 格式 (方括号)，严禁使用 Shell 格式
CMD ["python", "main.py"]
```

> **致命陷阱**：若写成 `CMD python main.py`，Docker 会以 `/bin/sh -c` 启动，`/bin/sh` 默认不转发信号给 Python，停机超时 10 秒后容器会被暴力 `SIGKILL` 杀死，导致数据正在落盘时损坏！

---

## 3. 生产级 HEALTHCHECK 探针标准

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://127.0.0.1:8000/healthz || exit 1
```

- `--interval=30s`：常规探测频率；
- `--timeout=5s`：探测单次超时；
- `--start-period=10s`：应用冷启动预热期，期间失败不累加；
- `--retries=3`：连续失败 3 次标记容器为 `unhealthy`。
