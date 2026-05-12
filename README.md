# iostiny · 公开技术文档

[![Live](https://img.shields.io/badge/Live-doc.iottiny.top-aa1141)](https://doc.iottiny.top/)

中文技术文档与图谱集合。所有内容 self-contained HTML + SVG，零依赖，浏览器双击即看。

> 在线访问：<https://doc.iottiny.top/>（GitHub Pages 兜底：<https://iostiny.github.io/doc/>）

## 当前专题

主入口 `index.html` 按三个分组组织：

### 01 · IM 安全通信

- 🔐 [**Signal Protocol · 中文图谱**](./signal-protocol/) — 四套讲法
  - [渐进版 · 25 步交互教程](./signal-protocol/beginner/)：零基础推进 Double Ratchet，每步附 Rust 骨架与权威外链
  - [深度版 · 11 张架构图谱](./signal-protocol/advanced/)：面向已熟悉 Signal 的工程师做 recall，含服务端契约
  - [Sesame · 多设备会话管理](./signal-protocol/sesame/)：Signal 多设备分发模型
  - [PQXDH · 后量子演进图谱](./signal-protocol/pqxdh/)：从 X3DH 到 PQXDH (2023) 再到 SPQR / Triple Ratchet (2025)，含 harvest-now-decrypt-later 威胁模型、KEM ≠ DH 心智迁移、hybrid 安全论证
- 🌀 [**Noise Protocol · 中文渐进图谱（Bitchat 视角）**](./noise-protocol/) — 以 Bitchat 实战切入 Noise Framework
  - [渐进版教程](./noise-protocol/beginner/)
- 🟢 [**Matrix · vodozemac 对照 Signal**](./matrix-protocol/) — 以 vodozemac 0.10.0 源码为切入点的 Matrix 加密栈（Olm + Megolm）与 Signal 对照图谱：3DH vs X3DH（含 fallback key 修正）、Megolm vs Sender Keys、联邦威胁面、2.5 年 PQ 差距、"PQ-Olm" 推演设计

### 02 · 区块链密码学

- 🧮 [**Zero-Knowledge Proofs · 中文渐进图谱**](./zkp/) — 零知识证明、承诺方案、Schnorr 等
  - [渐进版教程](./zkp/beginner/)

### 03 · AGENT / LLM

- 📄 [**AGENT 时代，Markdown 已死，HTML 当立**](./html-over-markdown/) — Thariq《Unreasonable Effectiveness of HTML》原文图谱与中文解读

## 仓库结构

```
doc/
├── index.html              ← 主入口 hub（三组导航）
├── signal-protocol/        ← IM 安全通信 · Signal
│   ├── index.html          ← 专题 landing
│   ├── beginner/           ← 25 步渐进教程
│   ├── advanced/           ← 11 张架构图
│   ├── sesame/             ← 多设备会话
│   └── pqxdh/              ← 后量子演进 (PQXDH + SPQR)
├── noise-protocol/         ← IM 安全通信 · Noise
│   ├── index.html
│   └── beginner/
├── matrix-protocol/        ← IM 安全通信 · Matrix (vodozemac 对照 Signal)
│   └── index.html
├── zkp/                    ← 区块链密码学
│   ├── index.html
│   └── beginner/
├── html-over-markdown/     ← AGENT / LLM
│   └── index.html
├── docs/                   ← 仓库内部 playbook（不部署给读者）
├── AGENTS.md               ← 给 AI 编码代理的项目指南
└── CNAME                   ← doc.iottiny.top
```

每个专题独立 self-contained，可单独下载离线查看。

## 技术栈

纯静态 HTML + SVG + 原生 JS + CSS Custom Properties。
不引入 npm、构建链、SPA 框架。
Google Fonts 是唯一外链。

视觉基线：cream 底（`#fdfaf6`）+ cherry red 主调（`#aa1141`）。字体 Noto Sans SC + JetBrains Mono。

## 贡献 / 二次创作

新增专题、修订内容前请先读 [`AGENTS.md`](./AGENTS.md)，其中包含分组规则、主 hub 的 `.topic-card` 模板、专题内容设计原则（slider / details / mobile）以及已知陷阱。

## License

代码 (HTML / CSS / JS / SVG)：MIT
文档与图示：CC-BY 4.0
