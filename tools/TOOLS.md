# Tools 工具目录

此目录包含 VisionC2 的构建、加密、部署和清理辅助工具。

## `setup.py`

项目根目录中的主安装向导。它负责：

- 生成 C2 地址配置、magic code、协议版本和 AES 密钥
- 生成 TLS 证书
- 加密 `bot/config.go` 中的敏感字符串
- 按模块选项构建 CNC、relay 和 14 架构 Bot

## `crypto.go`

独立加密/解密工具，使用与 Bot 相同的密钥。

```bash
go run tools/crypto.go encrypt <string>
go run tools/crypto.go encrypt-slice <a> <b> ...
go run tools/crypto.go decrypt <hex>
go run tools/crypto.go decrypt-slice <hex>
go run tools/crypto.go generate
go run tools/crypto.go verify
go run tools/crypto.go resetconfig
```

`resetconfig` 可将源码恢复为默认发布状态，便于重新构建。

## `build.sh`

交叉编译 14 种 Linux 架构的 Bot，并输出到项目根目录的 `bins/`。

| 二进制名 | 架构 | 说明 |
|----------|------|------|
| `ksoftirqd0` | x86 | 32 位 Intel/AMD |
| `kworker_u8` | x86_64 | 64 位 Intel/AMD |
| `jbd2_sda1d` | ARMv7 | Raspberry Pi 2/3 |
| `bioset0` | ARMv5 | 旧 ARM 设备 |
| `kblockd0` | ARMv6 | Raspberry Pi 1 |
| `rcuop_0` | ARM64 | 现代 ARM / SBC |
| `kswapd0` | MIPS | 大端路由器 |
| `ecryptfsd` | MIPSLE | 小端路由器 |
| `xfsaild_sda` | MIPS64 | 64 位 MIPS 大端 |
| `scsi_tmf_0` | MIPS64LE | 64 位 MIPS 小端 |
| `devfreq_wq` | PPC64 | PowerPC 大端 |
| `zswap_shrinkd` | PPC64LE | PowerPC 小端 |
| `edac_polld` | s390x | IBM System/390 |
| `cfg80211d` | RISC-V 64 | RISC-V 64 位 |

## `cleanup.sh`

用于清理本机上的 Bot 持久化痕迹。误运行 Bot 时可用 root 执行：

```bash
sudo bash tools/cleanup.sh
```

## `fix_botkill.sh`

CNC 服务端调优脚本，用于提升文件描述符限制、开放 443 端口并调整 TCP 缓冲区：

```bash
sudo bash tools/fix_botkill.sh
```

## `loader.sh`

部署 loader，会自动识别目标架构并下载匹配的 `bins/` 二进制。使用前需要修改脚本中的托管服务器地址。

## `upx`

随项目附带的打包器二进制，用于压缩 Bot 构建产物。