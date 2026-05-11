# iostiny · 公开技术文档

[![Live](https://img.shields.io/badge/Live-doc.iottiny.top-aa1141)](https://doc.iottiny.top/)

中文技术文档与图谱集合。所有内容 self-contained HTML + SVG，零依赖，浏览器双击即看。

> 在线访问：<https://doc.iottiny.top/>（GitHub Pages 兜底：<https://iostiny.github.io/doc/>）

## 当前专题

### 🔐 [Signal Double Ratchet · 双视角图谱](./signal-double-ratchet/)

当今主流 E2E 加密协议的核心。两套讲法：

- **🐣 [渐进版 · 25 步交互教程](./signal-double-ratchet/beginner/)** — 完全零基础，按 Next 一步一步推进，每步附 Rust 骨架代码与权威外链。
- **🎯 [深度版 · 11 张架构图谱](./signal-double-ratchet/advanced/)** — 面向已熟悉 Signal 协议的工程师做 recall，含服务端契约。

涵盖：ECDH on Curve25519、KDF 链、Symmetric Ratchet、DH Ratchet、Double Ratchet FSM、乱序处理、X3DH 异步握手、OPK 生命周期、Forward Secrecy + Post-Compromise Security。

## 仓库结构

```
doc/
├── index.html                 ← 主入口 (所有专题导航)
├── signal-double-ratchet/     ← Signal 专题
│   ├── index.html             ← 专题 landing
│   ├── beginner/              ← 25 步渐进教程
│   └── advanced/              ← 11 张架构图
└── (future) more-topics/      ← 未来扩展
```

每个专题独立 self-contained，可单独下载离线查看。

## 技术栈

纯静态 HTML + SVG + 原生 JS + CSS Custom Properties。
不引入 npm、构建链、SPA 框架。
Google Fonts 是唯一外链。

## License

代码 (HTML / CSS / JS / SVG)：MIT
文档与图示：CC-BY 4.0
