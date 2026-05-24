---
title: WebTerm —— 基于 Web 的运维管理工具
date: 2026-05-24
author: xufanchn
tags: [WebTerm, SSH, 运维, Go, React]
description: WebTerm 是一个基于 Web 的 SSH/SFTP/数据库运维管理工具，单二进制部署，浏览器即客户端。
---

# WebTerm —— 基于 Web 的运维管理工具

## 背景

在日常运维工作中，我们经常需要在不同机器之间切换，管理 SSH 连接、传输文件、查询数据库。传统的做法是使用 Putty、Xshell、WinSCP、Navicat 等多个客户端工具，不仅切换繁琐，而且配置难以同步。

**WebTerm** 应运而生 —— 一个基于 Web 的一站式运维管理工具，只需浏览器即可完成所有操作。

## 技术架构

WebTerm 采用前后端分离架构，但部署极简：

- **后端**：Go 语言编写，使用 `x/crypto/ssh` 处理 SSH 连接，`pkg/sftp` 处理文件传输，`go-sql-driver/mysql` 连接数据库
- **前端**：React + TypeScript，xterm.js 渲染终端，CodeMirror 6 提供代码编辑
- **存储**：SQLite 保存用户、连接配置、分组信息
- **部署**：前端静态文件通过 Go 的 `embed.FS` 打包进二进制文件，单文件即可运行

```
浏览器 (React SPA) ⟷ HTTP/WebSocket ⟷ Go 后端 (单二进制)
                                       ├── SSH 模块 (x/crypto/ssh)
                                       ├── SFTP 模块 (pkg/sftp)
                                       ├── 数据库模块 (go-mysql)
                                       └── 认证 + 配置 (SQLite)
```

## 核心功能

### SSH 终端

基于 xterm.js 的全功能终端模拟器：

- **多标签页**：同时打开多个 SSH 会话，标签支持拖拽排序
- **无限分屏**：横向或纵向拆分面板，每个面板独立标签栏
- **广播模式**：键盘输入同步发送到多个终端，批量操作利器
- **主题定制**：预设多套配色方案，支持自定义前景色、背景色、ANSI 色板
- **关键字高亮**：自定义关键字和颜色，ERROR/WARN/INFO/DEBUG 开箱即用
- **Zmodem 支持**：终端内 sz/rz 文件传输，自动触发浏览器文件对话框
- **心跳重连**：WebSocket 断连后指数退避重试，SSH 会话自动恢复

### SFTP 文件管理

WinSCP 风格的双栏文件管理器：

- **跟随模式**：SFTP 面板自动跟随当前终端的工作目录（通过 OSC 7 协议）
- **拖拽上传**：文件直接拖入浏览器，分块上传（10MB/块），底部进度条显示
- **在线编辑**：双击文本文件弹出 CodeMirror 6 编辑器，Ctrl+S 保存回远程
- **双栏传输**：左右面板可分别连接不同服务器，拖拽触发跨服务器传输

### 数据库管理

MySQL 可视化管理：

- **表浏览**：树形展示 数据库 → 表 → 列（类型、索引）
- **查询编辑器**：CodeMirror 6 + SQL 方言高亮，Ctrl+Enter 执行
- **内联编辑**：双击结果表格单元格直接修改数据
- **结果导出**：查询结果一键导出 CSV
- **SQL 格式化**：Ctrl+Shift+F 一键美化 SQL

### 安全设计

- SSH 凭据使用 **AES-256-GCM** 加密存储，密钥来自配置文件
- JWT 认证，24 小时过期
- DDL 操作需二次弹窗确认
- 服务端不记录完整 SQL 查询内容
- 连接按用户隔离，支持共享连接

## 快速安装

### 二进制部署

```bash
# 下载最新版本
wget https://github.com/xufanchn/webterm/releases/latest/download/webterm_linux_amd64.tar.gz
tar -xzf webterm_linux_amd64.tar.gz

# 创建配置文件
cat > config.yaml << EOF
port: 8080
encryption_key: $(openssl rand -hex 16)
EOF
chmod 600 config.yaml

# 启动
./webterm -config config.yaml
```

### Docker 部署

```bash
docker run -d \
  --name webterm \
  -p 8080:8080 \
  -v $(pwd)/config.yaml:/app/config.yaml \
  -v $(pwd)/data:/app/data \
  ghcr.io/xufanchn/webterm:latest
```

启动后访问 `http://localhost:8080`，默认账号 `admin/admin`，登录后请立即修改密码。

## 路线图

- [x] SSH 终端（多标签、分屏、广播、主题）
- [x] SFTP 文件管理（跟随模式、双栏、在线编辑）
- [x] MySQL 数据库管理（查询、浏览、内联编辑）
- [ ] LDAP/AD 集成认证
- [ ] PostgreSQL 支持
- [ ] 会话录制与回放
- [ ] 配置模板下发

## 结语

WebTerm 的核心理念是**简单部署，强大功能**。一个二进制文件，一个浏览器，就能管理所有远程服务器。如果你也厌倦了在多个工具之间切换，不妨试试 WebTerm。

项目开源在 [GitHub](https://github.com/xufanchn/webterm)，欢迎 Star 和贡献。
