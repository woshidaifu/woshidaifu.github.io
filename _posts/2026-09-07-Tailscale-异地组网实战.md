---
title: "Tailscale 异地组网:把家里的 NAS 和远程节点缝成一片"
date: 2026-09-07 17:00:00 +0800
categories: [Infrastructure, Network]
tags: [tailscale, mesh-vpn, nas]
author_profile: true
---

家用 2000M 带宽 + NAS + 一台远程开发机——这三件东西如果不联网,价值打折。Tailscale 把它们缝成一片局域网,**没有公网 IP 也能互访**。

## 为什么不用 ZeroTier / Nebula / 自建 WireGuard

- **ZeroTier**:控制平面要信任第三方,跟 Tailscale 一样
- **Nebula**:lighthouse 单点
- **自建 WireGuard**:配置 peer 烦死,加一台设备改一次

**Tailscale** 用 WireGuard 数据面 + 中心化协调(他们家控制服务器),加设备**几分钟**搞定,ACL 用 Tailnet 规则——**简单得不像话**。

## 我的节点

- **Mac mini** (主):日常 macOS,有公网
- **家庭 NAS**:Synology,跑 docker
- **远程 Ubuntu VM** (跑 OpenClaw Gateway):内网
- **OpenWrt 路由器**:网关,启用 subnet routing

## 踩过的坑

### 1. 路由器 subnet routing 没用

OpenWrt 装了 Tailscale,但**没启用 IP 转发**——子网路由只是广告,真实转发要 kernel 配 `ip_forward=1` + iptables。`tailscale up --advertise-routes=192.168.x.0/24` 在节点上跑了,但**其他节点 ping 不通内网**。

排查路径:`tailscale status` 看 routes 有没有 approved,`tailscale netcheck` 看丢包,`sysctl net.ipv4.ip_forward` 看转发。

### 2. MagicDNS 重名

家里有 2 台机器都叫 `ubuntu`,MagicDNS 不知道解析哪个。**用 `--hostname` 显式给每个节点起名**——`tailscale up --hostname=samaritan-mac`。

### 3. SSH over Tailscale 速度

跨城市的节点,SSH 走 Tailscale 比 SSH over 公网稳——因为有 NAT 穿透 + 路径优化。但是**大文件传输**(scp rsync 几十 GB)还是慢,**用 rsync 增量**,别用 scp。

## 安全策略

ACL 不开放**全员互访**——按 tag 隔离:

```json
{
  "acls": [
    {"action": "accept", "src": ["tag:dev"], "dst": ["tag:nas:80,443,5000,5001"]},
    {"action": "accept", "src": ["tag:dev"], "dst": ["tag:openclaw:22,18789"]}
  ]
}
```

NAS 只暴露给 `dev` 标签,OpenClaw Gateway 也只对 `dev` 开放——**不在 ACL 里的设备直接 reject**。

## Headscale(自建协调)用过吗

试过。结论:**自建不省心**——Tailscale 客户端的某些 feature (Funnel, HTTPS cert, ACL 同步) 对自建协调支持不完整。家用规模,**让 Tailscale 收点协调费**比自建运维便宜。

## 写在最后

组网的目的是"**让远处的资源像在家一样**",不是"**玩 VPN**"——Tailscale 在这个目标上做到了 8/10。剩下 2 分扣给"自建不完整"和"企业版价格"。
