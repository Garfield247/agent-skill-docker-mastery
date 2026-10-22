# 多阶段构建架构与 Layer 缓存黄金军规 (Multistage & Caching)

## 1. 变动频次由低到高排序原则 (Change Frequency Ordering)

Docker 构建严格利用分层缓存机制。编写 `Dockerfile` 时必须按照以下次序排列：

```text
1. 基础系统依赖与包管理器安装 (极少变动，排最前)
   ↓
2. 语言包依赖锁文件拷贝 (go.mod, requirements.lock, package-lock.json)
   ↓
3. 依赖包下载预热与静态编译 (变动频率低)
   ↓
4. 业务应用源代码拷贝 (COPY . .) (变动频率极高，排最后)
   ↓
5. 最终轻量编译与资产打包
```

---

## 2. 同一 Layer 缓存清理军规 (Clean Cache in Same RUN)

在同一个 `RUN` 指令中完成安装与清理，防止临时缓存膨胀成独立持久化层：

```dockerfile
# ✅ 正确：同一层内即刻清理 apt 缓存
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        tzdata \
    && rm -rf /var/lib/apt/lists/*

# ❌ 错误：分两行执行，缓存已固化在上一层，无法减小最终镜像体积
# RUN apt-get update && apt-get install -y ca-certificates
# RUN rm -rf /var/lib/apt/lists/*
```
