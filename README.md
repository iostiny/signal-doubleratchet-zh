# Signal Double Ratchet · 中文图谱

[![Pages](https://img.shields.io/badge/Live-iostiny.github.io-aa1141)](https://iostiny.github.io/signal-doubleratchet-zh/)

Signal Double Ratchet 协议的两套独立讲法：

- **🐣 [渐进版 · 25 步交互教程](./diagrams-beginner/)** — 完全零基础，按 Next 一步一步推进，每步附 Rust 骨架代码与权威外链。
- **🎯 [深度版 · 11 张架构图谱](./diagrams/)** — 面向已熟悉 Signal 协议的工程师做 recall，含服务端契约。

> 在线访问：<https://iostiny.github.io/signal-doubleratchet-zh/>

## 为什么做这个

官方 [Signal Protocol 规范](https://signal.org/docs/) 严谨，但对新手门槛较高；中文资料则普遍停留在「介绍性博客」层级，缺少把「直觉 → 数学 → 工程实现」串起来的渐进材料。

本仓库尝试：

- **新手版**用 25 个累积演进步骤，每步只引入一个新概念
- **老手版**用 11 张独立架构图，覆盖 KDF 链 / DH ratchet / 乱序处理 / X3DH / 服务端契约
- **双版本共用** `phycat-cherry`（cream + cherry red）视觉主题，长时间阅读不疲劳

## 技术栈

零依赖：纯静态 HTML + SVG + 原生 JS + CSS Custom Properties。浏览器双击即看。

## 内容范围

| 主题 | 新手版 | 老手版 |
|---|---|---|
| 加密动机 / 对称密钥 | ✅ | — |
| ECDH on Curve25519 | ✅ | ✅ |
| KDF 链 / Symmetric Ratchet | ✅ | ✅ |
| Forward Secrecy 直觉 | ✅ | ✅ |
| DH Ratchet · ping-pong | ✅ | ✅ |
| Double Ratchet FSM | ✅ | ✅ |
| 乱序消息处理 | — | ✅ |
| X3DH 异步握手 | ✅ | ✅ |
| OPK 池生命周期 | ✅ | ✅ |
| 服务端原子契约 | — | ✅ |
| Rust 实现骨架 | ✅ (9 个关键 step) | — |
| Sesame / 群组 / Sealed Sender | — | — |

## Rust 实现参考

新手版的 🦀 卡片基于 [RustCrypto](https://github.com/RustCrypto) 与 [dalek-cryptography](https://github.com/dalek-cryptography) 生态：

```toml
x25519-dalek    = "2"       # ECDH on Curve25519
hkdf            = "0.12"    # 派生 RK / CK / SK
hmac            = "0.12"    # KDF_CK 底层
sha2            = "0.10"    # SHA-256
aes-gcm         = "0.10"    # AEAD 消息加密
zeroize         = "1"       # 私钥/MK 用完清零
```

生产级参考实现：[`signalapp/libsignal`](https://github.com/signalapp/libsignal)

## License

代码 (HTML/CSS/JS/SVG)：MIT
文档与图示：CC-BY 4.0

Signal Protocol 本身 © Open Whisper Systems。
