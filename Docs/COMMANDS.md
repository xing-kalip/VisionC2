# VisionC2 命令与快捷键参考

> 中文速查版，保留原始命令名和快捷键，方便与源码和面板保持一致。

## 启动

```bash
./server          # 启动 TUI
./server --split  # 启动 Telnet CLI
./server --web    # 启动 Web 面板
```

## 全局快捷键

| 按键 | 作用 |
|------|------|
| `↑` / `k` | 上移 |
| `↓` / `j` | 下移 |
| `←` / `→` | 切换标签或视图 |
| `Enter` | 选择 / 确认 |
| `q` | 返回 / 退出 |
| `Esc` | 取消 / 退出当前模式 |
| `r` | 刷新 |

## Bot 列表

| 按键 | 作用 |
|------|------|
| `Enter` | 打开选中 Bot 的远程 Shell |
| `b` | 广播 Shell |
| `l` | 使用选中 Bot 启动任务 |
| `i` | 请求 `!info` |
| `p` | 发送 `!persist`，需要确认 |
| `r` | 发送 `!reinstall`，需要确认 |
| `k` | 移除并终止，需要确认 |

## Shell

| 命令 | 说明 |
|------|------|
| `!shell <cmd>` | 执行命令并返回输出 |
| `!detach <cmd>` | 后台执行，不等待输出 |
| `!stream <cmd>` | 实时流式输出 |
| `!info` | 获取系统信息 |
| `!persist` | 安装持久化 |
| `!reinstall <url>` | 从 URL 重新安装 |
| `!download <path>` | 下载文件 |
| `!upload <path> <base64>` | 上传文件 |

## SOCKS

| 命令 | 说明 |
|------|------|
| `!socks relay:port` | 通过 relay 反连启动 SOCKS5 |
| `!socks 1080` | 直连模式，在 Bot 上监听本地端口 |
| `!stopsocks` | 停止代理 |
| `!socksauth <user> <pass>` | 更新 SOCKS5 认证 |

## Web 面板快捷键

| 按键 | 作用 |
|------|------|
| `1` | Bot 标签 |
| `2` | SOCKS 标签 |
| `3` | Attack 标签 |
| `4` | Activity 标签 |
| `5` | Tasks 标签 |
| `6` | Users 标签 |
| `/` | 聚焦搜索框 |
| `?` | 显示快捷键帮助 |
| `Esc` | 关闭弹窗或覆盖层 |

## Split / Telnet CLI

连接：

```bash
nc YOUR_SERVER 420
```

认证流程：

1. 输入触发词：`spamtec`
2. 输入用户名和密码

常用命令：

| 命令 | 说明 |
|------|------|
| `help` | 显示菜单 |
| `methods` | 列出可用方法 |
| `bots` | 列出在线 Bot |
| `ongoing` | 查看正在运行的任务 |
| `clear` / `cls` | 清屏 |
| `logout` / `exit` | 断开连接 |

## 备注

- 危险操作会要求确认
- Web 面板通过 SSE 实时刷新
- Web Shell 使用 WebSocket，并支持多标签会话
- Bot 超时后会自动清理离线状态