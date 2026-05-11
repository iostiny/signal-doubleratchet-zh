# AGENTS.md — Signal Double Ratchet 中文图谱

给在本仓库工作的 AI 编码代理（Claude Code / Cursor / Aider 等）的项目指导。

## 项目定位

两套互补的 Signal Double Ratchet 协议讲解：

- `diagrams-beginner/` — **零基础渐进版**：单页 HTML，25 步交互教程，按 Next 按钮累积演进，每步附 Rust 实现骨架 + 权威外链。
- `diagrams/` — **老手深度版**：11 张独立架构图谱 + 总览 `index.html`，每张图一个角度（KDF 链 / DH ratchet / X3DH / 服务端契约 / 乱序处理）。
- `index.html`（仓库根）— landing 页面，并列展示两条路径，与 phycat-cherry 主题视觉一致。

部署：GitHub Pages 直接 serve main branch root，URL `https://iostiny.github.io/signal-doubleratchet-zh/`。

## 技术栈与硬约束

- **零依赖**：纯静态 HTML + SVG + 原生 JS + CSS Custom Properties。**不引入** npm、构建链、SPA 框架、外部 JS 库。Google Fonts 是唯一外链。
- **浏览器双击即看**：所有路径必须是相对路径，不能依赖任何本地服务器。
- **每页 self-contained**：`diagrams-beginner/index.html` 单文件 1500+ 行，所有 SVG / CSS / JS 都内联；`diagrams/*.html` 同理。不抽公共 CSS 文件。
- **视觉主题**：phycat-cherry（cream `#fdfaf6` 底 + cherry red `#aa1141` 主调）。所有调色板变量定义在 `:root`，新代码必须复用 var(--cherry / --teal / --amber / --green / --violet / --slate)。
- **字体**：Noto Sans SC（中文）+ JetBrains Mono（代码 / 公式）。

## 渐进版（`diagrams-beginner/`）的累积演进机制

核心是 `data-step` + `data-step-end` 两个 SVG 属性 + JS `goTo(n)` 控制器。

```html
<g data-step="5" data-step-end="5">  <!-- 只在 step 5 显示 -->
<g data-step="5">                    <!-- step 5 开始出现，之后累积保留 -->
<g data-step="5" data-step-end="9">  <!-- step 5-9 显示，step 10 起隐藏 -->
```

JS 在 `goTo(n)` 里遍历所有 `[data-step]` 元素，根据 `n >= start && n <= end` 切 `.visible` class，配合 CSS `opacity + translateY` transition 实现淡入效果。

**章节 SVG 切换**：每章 (`<svg class="chapter-svg" data-ch="N">`) 在切到该章时整体 display:block，其他章 display:none。`goTo()` 同时更新 chapter SVG 和 element visibility。

### 添加 / 改 step 的 checklist

1. **加 step**：同步在 `STEPS` 数组（caption 文案）、`STEP_EXTRAS` 对象（refs + code）、对应章节 SVG 里加 `<g data-step="N">` 块。三者步数必须一致（当前 25）。
2. **判断累积 vs 限定**：
   - 结构性元素（CK 链节点、Alice/Bob 头像、泳道）— 一般用裸 `data-step="N"`，让后续 step 继承上下文。
   - 标题 / 副标题 / 顶部 hint / 红色警告 callout — **必须**用 `data-step="N" data-step-end="N"`，否则下一 step 加新标签时一定撞。
3. **检查相邻 step 是否撞**：上一 step 的 y=60 / y=100 / y=120 区域的文字，是否跟下一 step 新加的 y=124 / y=130 撞？经验：相邻 step 之间在同一 Y 区域 (±20px) 放文字几乎必撞，加 `data-step-end` 让旧文字消失。
4. **renderExtras 占位**：没 code 的 step 自动隐藏 🦀 卡片（不渲染占位文字），左侧 refs 卡占满整宽；这是 `extras.classList.toggle('two-col', ...)` 控制的。

## 视觉规范（角色色 / 坐标）

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

## Rust 骨架代码规范（`STEP_EXTRAS[N].code`）

放在 HTML 字符串里通过 `innerHTML` 注入到 `<pre>`，所以：

- **必须用 HTML 实体**：`<` 写成 `&lt;`，`>` 写成 `&gt;`。否则 `Vec<u8>` 这种泛型会被解析成 tag 标签。
- **语法着色用 span class**：`kw`（关键字 violet）、`ty`（类型 teal）、`fn`（函数名 cherry）、`cm`（注释 灰 italic）、`str`（字符串 amber）。CSS 在 `.extras-card.code .kw` 等已定义。
- **保持 10-15 行内**：教学骨架，能编译但不一定完整；目的是让读者直接复制起步。
- **依赖标注在第一行注释**：`// Cargo.toml:  x25519-dalek = "2"`。
- **避免 unsafe / 复杂泛型 / generic bounds**：新手会被劝退。
- **生态对齐**：用 [RustCrypto](https://github.com/RustCrypto) 和 [dalek-cryptography](https://github.com/dalek-cryptography) 系列 crate（x25519-dalek、hkdf、hmac、sha2、aes-gcm、zeroize）。生产参考 `signalapp/libsignal`。

## 老手版（`diagrams/`）的约定

- 每个文件 (`01-godview.html` … `11-key-cheatsheet.html`) 是独立 HTML，**不共享 JS 状态**，可单独打开。
- `index.html` 是导航总览，按 Foundation / Mechanics / X3DH / Server-side 4 个 section 分类。
- **`diagrams/index.html` 顶部那块 🐣 CTA 卡片**链接到 `../diagrams-beginner/`，是历史遗留；新仓库根 `index.html` 已经是统一入口，但这个 CTA 保留（让用户从老手版意外进入也能切换到新手版）。
- 视觉规范：与渐进版一致；但每张图额外有底部「📚 基础知识速查 · Reference」卡，是老手版的特色。

## 部署 & 维护

- **GitHub Pages**：source = main branch / root。push 后约 30-60s 自动构建发布。
- **检查 build 状态**：`gh api repos/iostiny/signal-doubleratchet-zh/pages/builds/latest --jq '.status'`，期望 `"built"`。
- **不要 push 的内容**：`.gitignore` 已忽略 `.DS_Store / .handoffs/ / tasks/`。PRD (`tasks/prd-*.md`) 在原工作目录维护，不上 repo。
- **原工作目录同步**：`~/Work/IMProject/Doc/P2P EncryptDecrypt/diagrams[-beginner]` 是 symlink，指向本 repo 对应目录。在原路径编辑等同于直接改 repo，省去 cp。

## 已知陷阱（被踩过的，别再踩）

1. **`~/.gitignore_global` 屏蔽 README.md / AGENTS.md / CLAUDE.md / settings.json** 等"AI 生成物"。`git add .` 会静默跳过这些文件 —— **必须用 `git add -f`**。
2. **`<svg>` 里直接写 `Vec<u8>` 会被解析成 tag**：HTML innerHTML 注入 Rust 代码时务必转义 `<` `>`。
3. **跨 step 文字撞**：在同一 y 区域加新文字而上一 step 的文字没 `data-step-end`，肉眼可见叠在一起。每加新文字先检查上一 step 同区域。
4. **emoji 占用很高**：`<text font-size="28">💀</text>` 实际向上扩展约 14px。文字标签放在 emoji 同一 y 的两侧会撞，把文字放到 y=124 / emoji 在 y=158 这种 30px 间隔才安全。
5. **kill MCP server 进程会让 Claude Code 断开工具**：`kill -9` 卡住的 chrome-devtools-mcp 不会触发自愈，反而会让所有 chrome 工具从工具表移除。要 reset MCP server 用 `/reload-plugins` 或重启 Claude Code，不要直接 kill。

## 不要做的事

- ❌ 引入 npm / 任何构建步骤
- ❌ 把 `diagrams-beginner/index.html` 拆成多文件（教学单文件 + URL hash 是核心 UX）
- ❌ 修改 phycat-cherry 主题色板（已和 12 张老手图全部统一，改色板影响巨大）
- ❌ 在渐进版加超过 25 步内容（节奏已经精心调校；新内容应该开新章而不是塞到现有 25 步里）
- ❌ 给概念性 step 强加 Rust 代码（占位文字已显式去除，没 code 就让卡片不显示更干净）
