# VisionC2 安装指南

> 这是 VisionC2 的中文安装与配置说明。`setup.py` 会自动处理配置生成、加密、源码补丁和构建流程。

## 前置要求

| 组件 | 最低要求 | 推荐配置 |
|------|----------|----------|
| 内存 | 512MB | 2GB+ |
| 存储 | 1GB | 5GB+ |
| 系统 | 任意 Linux | Ubuntu 22.04+ / Debian 12+ |
| 网络 | 443 端口开放 | 额外开放管理端口 |

安装依赖：

```bash
sudo apt update && sudo apt install -y openssl git wget gcc python3 screen netcat
wget https://go.dev/dl/go1.24.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && source ~/.bashrc
go version
```

## 初始化

```bash
git clone https://github.com/xing-kalip/VisionC2.git && cd VisionC2
python3 setup.py
```

选择 **[1] 完整初始化**。向导会询问 C2 地址、管理端口、TLS 证书信息和 Bot 模块配置。

## 启动方式

```bash
./server              # 默认交互模式
./server --split      # Telnet 管理模式
./server --web        # Web 面板
./server --daemon     # 无界面后台模式
```

后台运行：

```bash
screen -S vision ./server
```

首次运行会生成 root 用户和随机密码，请保存控制台中显示的密码。

## SOCKS5 中继部署

建议将 `relay_server` 部署在独立 VPS 上：

```bash
./relay_server
./relay_server -stats 127.0.0.1:9090
./relay_server -cp 9001 -sp 1080
```

默认端口：

| 端口 | 用途 |
|------|------|
| 9001 | Bot 反连控制端口 |
| 1080 | SOCKS5 客户端端口 |

详细说明见 [`PROXY.md`](PROXY.md)。

## TUI 操作概览

| 按键 | 作用 |
|------|------|
| `↑` / `↓` 或 `k` / `j` | 菜单导航 |
| `Enter` | 选择 |
| `q` / `Esc` | 返回或取消 |
| `r` | 刷新 |

常用视图：

- **Bot 管理**：查看在线 Bot、进入 Shell、执行持久化或移除
- **攻击启动器**：选择方法、目标、端口和持续时间
- **SOCKS 管理器**：启动中继或直连模式
- **连接日志**：查看 Bot 上下线记录

## 配置管理

敏感字符串会写入 `bot/config.go` 并通过 AES-CTR 加密。可使用 `tools/crypto.go` 进行验证或重新生成：

```bash
go run tools/crypto.go verify
go run tools/crypto.go generate
go run tools/crypto.go resetconfig
```

## 常见问题

| 问题 | 处理方式 |
|------|----------|
| 443 端口权限不足 | `sudo setcap 'cap_net_bind_service=+ep' ./server` |
| Bot 无法连接 | 检查 `ufw allow 443/tcp`、C2 地址和 TLS 连通性 |
| 构建失败 | 确认 Go 版本为 1.24+，并检查 `tools/build.sh` 输出 |
| 性能不足 | 在 CNC 服务器上运行 `sudo bash tools/fix_botkill.sh` |

## 安全与合规

- 仅在授权范围内测试
- 建议将 CNC 和 relay 分离部署
- 定期轮换中继基础设施
- 保留测试记录并遵守当地网络安全法律