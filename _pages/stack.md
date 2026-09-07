---
title: "Stack"
layout: single
permalink: /stack/
author_profile: true
---

## 🧰 AI 基础设施

| 组件 | 角色 |
|---|---|
| **OpenClaw** (Node) | Gateway,直连渠道交互 |
| **Hermes** (Python) | 并行辅助 agent,快速响应 + Web 搜索 |
| **LiteLLM** (Python) | 模型网关,统一管理多 provider 路由 |
| **Claude CLI / OpenCode** | 代码引擎 |
| **OpenWebUI / Cherry Studio** | 交互前端 |

## 🌐 网络与组网

| 工具 | 用途 |
|---|---|
| **Tailscale** | 异地组网,2000M 家庭带宽 mesh |
| **NAS** | 中央存储 |
| **OrbStack** | macOS 上的 Linux VM 运行时 |

## 🖥️ 操作系统与运行时

- **Mac mini** (arm64):日常环境,有外网
- **Ubuntu VM** (OrbStack):OpenClaw / LiteLLM 生产
- **离线内网环境** (x86_64, air-gapped):生产与办公,使用本地 Docker 镜像
- **Node.js**:nvm 管理(v26 LTS),离线环境锁 v18
- **Python**:严格 venv 隔离

## 💰 投资与数据

- **finance-mcp**:估值引擎(理杏仁商业 API)
- **三脚架架构**:宽基 / 红利 / 黄金
- **每日复盘**:cron 推送 + Telegram topic 67

## 📝 知识管理

- **Obsidian Vault**:110+ Markdown,按功能域划分(AI/Network/Coding/...)
- **Dreaming 机制**:每日 03:00 短期回忆提权 + 长期蒸馏
- **MEMORY.md**:curated 长期记忆,主 Gateway 加载

## 🛠️ 开发哲学

> 稳定性 > 新颖性。追新版本,但不为"预防"堵住升级路径。
