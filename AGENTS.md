# AGENTS.md — iostiny 公开技术文档

给在本仓库工作的 AI 编码代理（Claude Code / Cursor / Aider 等）的项目指南。

## 仓库定位

`doc` 是 iostiny 维护的**多专题中文技术文档集合**，部署在 `https://doc.iottiny.top/`（GitHub Pages，源 = main branch root）。

```
doc/
├── index.html                 ← 主入口 hub：列出所有专题卡
├── README.md / AGENTS.md / CLAUDE.md / CNAME / .gitignore
└── signal-double-ratchet/     ← 当前唯一专题
    ├── index.html             ← 专题 landing
    ├── beginner/              ← 25 步渐进教程
    └── advanced/              ← 11 张架构图
```

未来加新专题就是 `mkdir <topic-slug>/` + 在主 `index.html` 加一张卡。每个专题完全独立，可单独下载离线查看。

## 全仓通用约束（所有专题必须遵守）

- **零依赖**：纯静态 HTML + SVG + 原生 JS + CSS Custom Properties。**不引入** npm、构建链、SPA 框架、外部 JS 库。Google Fonts 是唯一外链。
- **浏览器双击即看**：所有路径必须是相对路径，不能依赖任何本地服务器。
- **每页 self-contained**：单文件可以很长（1500+ 行也接受），所有 SVG / CSS / JS 都内联；不抽公共 CSS 文件、不在专题之间共享 JS。
- **视觉主题**：cream 底（`#fdfaf6`）+ cherry red 主调（`#aa1141`）。所有调色板变量定义在 `:root`，新代码必须复用 `var(--cherry / --teal / --amber / --green / --violet / --slate)`。每个专题可以扩展自己的语义色，但基色必须保持一致。
- **字体**：Noto Sans SC（中文）+ JetBrains Mono（代码 / 公式）。
- **每个专题根目录必须有 `index.html`**：作为该专题的 landing 页，列出该专题下所有子内容。

## 添加新专题的标准流程

1. 选 slug（`kebab-case`，例如 `zkp-tutorial`、`tls-1.3-notes`）
2. `mkdir <slug>/` + 写专题 landing `<slug>/index.html`（参考 `signal-double-ratchet/index.html` 的两栏布局）
3. 在仓库根 `index.html` 的 `.topic-grid` 里加一张 `<a class="topic">` 卡片
4. 视觉沿用 cherry palette + 顶部 chip + 描述 + 元数据
5. 内部子目录命名建议用语义名（`beginner/` / `advanced/` / `reference/`），不要用实现细节命名（避免 `diagrams/`、`html-files/`）

## Signal Double Ratchet 专题（`signal-double-ratchet/`）

当前唯一专题。Signal Protocol 的中文渐进图谱。两个子目录：

- `signal-double-ratchet/beginner/index.html` — 单页 25 步交互教程
- `signal-double-ratchet/advanced/index.html` — 11 张独立架构图的总览 + `01..11-*.html` 11 个文件

### 累积演进机制（`beginner/index.html` 的核心）

`data-step` + `data-step-end` 两个 SVG 属性 + JS `goTo(n)` 控制器：

```html
<g data-step="5" data-step-end="5">  <!-- 只在 step 5 显示 -->
<g data-step="5">                    <!-- step 5 开始出现，之后累积保留 -->
<g data-step="5" data-step-end="9">  <!-- step 5-9 显示，step 10 起隐藏 -->
```

JS 在 `goTo(n)` 里遍历所有 `[data-step]` 元素，根据 `n >= start && n <= end` 切 `.visible` class，配合 CSS `opacity + translateY` transition 实现淡入。章节 SVG (`<svg class="chapter-svg" data-ch="N">`) 在切到该章时整体 display:block。

### 添加 / 改 step 的 checklist

1. **加 step**：同步在 `STEPS` 数组（caption 文案）、`STEP_EXTRAS` 对象（refs + code）、对应章节 SVG 里加 `<g data-step="N">` 块。三者步数必须一致（当前 25）。
2. **判断累积 vs 限定**：
   - 结构性元素（CK 链节点、Alice/Bob 头像、泳道）— 一般用裸 `data-step="N"`，让后续 step 继承上下文。
   - 标题 / 副标题 / 顶部 hint / 红色警告 callout — **必须**用 `data-step="N" data-step-end="N"`，否则下一 step 加新标签时一定撞。
3. **检查相邻 step 是否撞**：上一 step 的 y=60 / y=100 / y=120 区域的文字，是否跟下一 step 新加的 y=124 / y=130 撞？经验：相邻 step 之间在同一 Y 区域 (±20px) 放文字几乎必撞，加 `data-step-end` 让旧文字消失。
4. **renderExtras 占位**：没 code 的 step 自动隐藏 🦀 卡片（不渲染占位文字），左侧 refs 卡占满整宽；这是 `extras.classList.toggle('two-col', ...)` 控制的。

### 视觉规范（角色色 / 坐标）

| 角色 | CSS 变量 | 用途 |
|---|---|---|
| Alice | `--cherry` | 客户端发起方 |
| Bob | `--teal` | 客户端接收方 |
| Eve | `--red-deep` | 攻击者 / 偷听者 |
| Server | `--violet` | 服务端、密钥仓库 |
| Root chain | `--amber` | RK 主时间线 |
| Sending chain | `--cherry` | Alice 发送链 |
| Receiving chain | `--teal` | Alice 接收链 |

SVG viewBox 统一 `0 0 1100 520`。顶部 hint text 一般在 y=60；中间主区 y=80-460；底部说明在 y=430-490。

### Rust 骨架代码规范（`STEP_EXTRAS[N].code`）

放在 HTML 字符串里通过 `innerHTML` 注入到 `<pre>`，所以：

- **必须用 HTML 实体**：`<` 写成 `&lt;`，`>` 写成 `&gt;`。否则 `Vec<u8>` 这种泛型会被解析成 tag 标签。
- **语法着色用 span class**：`kw`（关键字 violet）、`ty`（类型 teal）、`fn`（函数名 cherry）、`cm`（注释 灰 italic）、`str`（字符串 amber）。CSS 在 `.extras-card.code .kw` 等已定义。
- **保持 10-15 行内**：教学骨架，能编译但不一定完整；目的是让读者直接复制起步。
- **依赖标注在第一行注释**：`// Cargo.toml:  x25519-dalek = "2"`。
- **避免 unsafe / 复杂泛型 / generic bounds**：新手会被劝退。
- **生态对齐**：用 [RustCrypto](https://github.com/RustCrypto) 和 [dalek-cryptography](https://github.com/dalek-cryptography) 系列 crate（x25519-dalek、hkdf、hmac、sha2、aes-gcm、zeroize）。生产参考 `signalapp/libsignal`。

### `advanced/` 子目录约定

- 每个文件 (`01-godview.html` … `11-key-cheatsheet.html`) 是独立 HTML，**不共享 JS 状态**，可单独打开。
- `advanced/index.html` 是该子集的导航总览，按 Foundation / Mechanics / X3DH / Server-side 4 个 section 分类。
- 顶部 🐣 CTA 卡片链接到 `../beginner/`，让从老手版进入的读者也能切到新手版。
- footer 含返回 Signal 专题首页 + 跳到主入口的链接。
- 每张图额外有底部「📚 基础知识速查 · Reference」卡。

## 部署 & 维护

- **GitHub Pages**：source = main branch / root。push 后约 30-60s 自动构建发布。
- **自定义域名**：`CNAME` 文件内容是 `doc.iottiny.top`。DNS 由 Cloudflare 管理（NameServer = `*.ns.cloudflare.com`）。Cloudflare DNS record：`CNAME doc → iostiny.github.io`，**Proxy 必须为 DNS only**（灰色云），否则与 GitHub Pages 的 Let's Encrypt 证书冲突。
- **检查 build 状态**：`gh api repos/iostiny/doc/pages/builds/latest --jq '.status'`，期望 `"built"`。
- **`.gitignore` 已忽略** `.DS_Store / .handoffs/ / tasks/`。

## 已知陷阱（技术层面）

1. **`<svg>` 里直接写 `Vec<u8>` 会被解析成 tag**：HTML innerHTML 注入 Rust 代码时务必转义 `<` `>` 为 HTML 实体。
2. **跨 step 文字撞**：在同一 y 区域加新文字而上一 step 的文字没 `data-step-end`，肉眼可见叠在一起。每加新文字先检查上一 step 同区域。
3. **emoji 占用很高**：`<text font-size="28">💀</text>` 实际向上扩展约 14px。文字标签放在 emoji 同一 y 的两侧会撞，把文字放到 y=124 / emoji 在 y=158 这种 30px 间隔才安全。
4. **`README.md` 等"AI 生成物白名单"**：有些开发者的 `~/.gitignore_global` 会默认屏蔽 `README.md / AGENTS.md / CLAUDE.md`。若 `git add .` 后这些文件没出现在 `git status`，用 `git add -f` 强制添加。

## 不要做的事

- ❌ 引入 npm / 任何构建步骤
- ❌ 把单页教学（如 beginner/index.html）拆成多文件（单文件 + URL hash 是核心 UX）
- ❌ 修改主题色板（所有专题视觉统一在 cream + cherry red，改色板影响巨大）
- ❌ 给概念性 step 强加 Rust 代码（占位文字已显式去除，没 code 就让卡片不显示更干净）
- ❌ 跨专题共享 JS / CSS 文件（每个专题必须 self-contained）
