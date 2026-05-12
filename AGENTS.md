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
- **不要在页面里自夸技术栈**：诸如"self-contained HTML + SVG · 无依赖、无构建 · 浏览器双击即看 · 浅色主题 cream + cherry red · URL hash 自动同步可分享"这种描述属于站点级约束，写在本 AGENTS.md 即可，**不要复制到任何专题 landing / sub-page 的 intro / hero-meta 段落里**——读者看到的是内容，不是工程介绍。短 chip 标签（如 hero-meta 里的 `零依赖 · self-contained`）可以保留，但完整描述句必须从 HTML 里去掉。

## 主 Hub (`index.html`) 设计规范

主入口是 **content-driven** 风格（不是 portfolio、不是 corporate grid）。试过 portfolio "hi, 我是 iostiny" 风格 —— 用户反馈"像简历不像文档站"，已废弃。**禁止再回到 portfolio 写法**。

### 四个区块（自上而下）

1. **Hero** — 极简，无 bio，无 hello-style
   - 左：`iostiny / doc` 用 JetBrains Mono 字体，`/` 用 `--cherry-pink` 色
   - 右（推到 main 末端）：一行灰色 tagline，如"中文技术文档与图谱"
   - 下方 1px 浅灰分隔线
2. **专题卡片**（核心）— 每张专题一张大卡 (`.topic-card`)，上下两段结构（见下）
3. **Upcoming hint** — 单行 mono 灰字 `下一个专题在路上 · X · Y · Z · ……`
   - **绝对不要**做"占位卡"（虚线框 + "更多专题陆续添加"）—— 视觉上强调"啥都没有"
4. **Footer** — 一行 mono：`github.com/iostiny/doc · CC-BY 4.0 · 源码 MIT`

### `.topic-card` 的结构

每张卡 `<a class="topic-card" href="./<slug>/">` 上下两段：

**上半 `.preview`**（高约 170px）
- 浅米黄渐变背景（`linear-gradient(135deg, #fffaf2 0%, #ffffff 100%)`），hover 时变 `#fff5ea → #fff`
- 内嵌一个 `<svg viewBox="0 0 760 170">`，必须是**该专题里真实存在的可视化的缩略版**，不是装饰图、不是 emoji、不是 generic icon
- 选最具识别度的视觉。Signal 选了 KDF 链（CK→KDF→CK 派生 MK），因为它是 Symmetric Ratchet 的核心标志
- 顶部 11px 小灰字描述（如 `Signal · Symmetric Ratchet · 每条消息派生专属 MK`）
- 底部 `.preview-caption`：`PREVIEW · TOPIC NAME` mono 小字，let-spacing 0.04em

**下半 `.card-body`**
- `.card-tag` — `SIGNAL · E2EE · 协议图谱` 类型 chip，cherry-glass 底
- `.card-title` — 1.3rem cherry-deep h2
- `.card-desc` — 0.94rem 灰字描述，max 3-4 行（max-width 720px）
- `.card-bullets` — 2×2 网格（CSS grid `repeat(auto-fit, minmax(200px, 1fr))`），4 条简短要点，cherry-pink 圆点
- `.card-cta` — `开始阅读 →`，cherry 边框；hover 整卡时反色为实心 cherry + 箭头右移 4px

### 配色策略（重要）

- **Preview 区**：用**该专题的"标志性配色"**，让访客一眼记住"这个专题长什么样"
  - Signal: `--amber`（CK / 链）+ `--cherry`（MK / 消息）—— 跟教程内部一致
  - ZKP (未来): 建议 `--violet`（commitment）+ `--green`（verify）
  - TLS (未来): 建议 `--teal`（handshake）+ `--cherry`（cipher）
- **Card body**: 永远用 `--cherry`（site 主色），跨专题统一
- 视觉语义："preview 是内容的色，card body 是 site 的色"

### 整卡可点 + hover 动画

整张 `.topic-card` 是 `<a>` 元素。hover 时：
- 卡片 `translateY(-3px)` + 加深 box-shadow + cherry 边框
- preview-bg 微变（更暖）
- CTA 反色 + arrow 右移
- 全部 transition 0.18-0.2s ease

## 添加新专题的 checklist

按顺序执行：

1. **选 slug**：`kebab-case`，名词性，避免实现细节。例：`zkp-tutorial` ✓ / `zkp-htmls` ✗
2. **写专题内容**：`mkdir <slug>/` → 至少有一个 `<slug>/index.html` 作为专题 landing；若有多种讲法可分 `<slug>/beginner/` `<slug>/advanced/` 等语义子目录（参考 Signal 专题）
3. **挑 hero SVG**：在专题内容里选**最具标识度**的一张图，缩略为 760×170 viewBox。如果没有现成的，先做这张图再上线
4. **复制 `.topic-card` 块到主 `index.html`**：
   - 改 `href="./<slug>/"`
   - **重写 SVG preview**：保留 viewBox 760×170 和顶部小字 + 底部 caption 结构；图内主体换成新专题的标志性可视化；颜色用该专题的标志性配色
   - 改 `.card-tag` 为该专题的类型标签（如 `ZKP · COMMITMENT · 入门`）
   - 改 `.card-title` 为专题完整名（保留 `· 副标题` 格式）
   - 改 `.card-desc` 3-4 行简介
   - 改 `.card-bullets` 4 条要点（保持 2×2，每条约 10-16 字）
5. **更新 upcoming hint**：从未来列表里**移除**这个专题
6. **不要**改 hero、不要改 footer、不要改 CSS 变量、不要给主 hub 加新 section（如分组头）—— 只有专题 >= 3 个时才考虑分组
7. 测试 hover、CTA、mobile 响应式（≤600px）

### 主 hub 永远只有一个文件

`index.html` 单文件包含所有 CSS、SVG、JS。增加专题不创建新 HTML 文件，只在主 hub 里新增一张 `.topic-card`。

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

## SVG 撞图 / 覆盖问题排查（chrome-devtools-mcp 工作流）

**写完 step 不算完，要在浏览器里逐步走一遍**。SVG 静态看代码很难发现累积演进的撞图，必须实跑。下面是经过几次迭代验证的稳定流程：

### 标准排查工作流

1. **跑 mcp，逐步截图**：用 `chrome-devtools-mcp` 打开 `file:///.../<topic>/<sub>/index.html#step-N`，对**每个章节的边界 step**（章节末步 + 下一章首步）+ **每个引入大块新元素的 step** 截图。不要只看 Step 1 和 Step N，重点看"刚刚出现新元素"和"刚刚清屏"的步。同时 `list_console_messages(types=['error','warn'])` 检查 JS 是否报错。
2. **对照视觉问题分类**：
   - 文字叠在一起 → 上一 step 的文字没 `data-step-end`，新 step 在同 y 区域加了新文字
   - 框/形状叠在一起 → 上一 step 的图形元素没 step-end，新 step 在重叠 x/y 区域加新框
   - 旧概念应该消失但还在 → "替换型"展示没用 step-end 做交接（例：Step 5 的新 active/inactive 摞框替代 Step 3 的 SessionRecord 框，Step 3 必须 `data-step-end="4"`）
   - 标签描述跟当前 step 语义不符 → 上一 step 给某元素配的副标在新 step 还显示，但语义已变（例：Step 11 的 Server 副标 "key distribution" 在 Step 12 stale 场景里就变成误导）
3. **定位代码**：`rg -n 'data-step="N"' <file>.html` 列出该 step 所有元素，再 `Read` 看这些 group 的 `<rect> <text> <line>` 坐标。
4. **算重叠**：拿出涉嫌撞图的两个元素的 `x / y / width / height`，心算 bounding box 是否相交。常见场景：viewBox 0-1100 × 0-520，新元素在 y=380-500，旧元素在 y=370-420 → 必撞。
5. **选修复策略**（按优先级）：
   - **加 `data-step-end="N"` 限定可见窗口** —— 适用于"hint 文字 / 警告 callout / 替换型展示"
   - **拆分原 step group**：把"持续元素"（如 actor 圆 / Server 椭圆）和"step-only 元素"（如 OutEnvelope 详情 box）拆成两个 group，前者裸 `data-step="N"`，后者加 `data-step-end="N"`
   - **重新选坐标**：新元素挪到空白区。优先挪而不是缩，缩文字会丢可读性
   - **改副标使其语义跨 step 通用**：例 "key distribution" → "目录服务"，覆盖 prekey + device list 两种用途
6. **用 `evaluate_script` 验证假设**：截图模糊时，跑一段 JS 拿到精确的 bounding box / class / data-* 属性，比肉眼准。例：
   ```js
   () => Array.from(document.querySelectorAll('.chapter-svg.active rect'))
     .map(r => ({ x: r.getAttribute('x'), y: r.getAttribute('y'),
                  w: r.getAttribute('width'), h: r.getAttribute('height'),
                  stepEnd: r.closest('[data-step]')?.dataset.stepEnd }))
   ```
7. **修完 reload 再截图**：`navigate_page(type='reload', ignoreCache=true)`，**不要相信缓存**。截图前后对比，确认问题消失且没引入新问题。

### 加新 step 之前的预防 checklist

- 上一 step 在同 y 区域 (±30px) 是否有不该保留的元素？没 step-end 就加 step-end。
- 新加的"概念性面板/算法 box"是否撞了已有的 Server / 表格 / 长条？先在脑子里画 bounding box 再写坐标。
- 新加的"替换型"元素（旧概念被新表达替代）—— 旧元素的 group 必须加 `data-step-end="<新 step - 1>"`。
- 顶部 hint 文字必须 `data-step-end="N"`（每步只活一步），不能裸 `data-step="N"`。

### 不要这样排查

- ❌ 不要只看代码就断定"应该没问题"。SVG 累积演进的撞图必须实跑才能发现。
- ❌ 不要用 sleep 等 GitHub Pages 部署后再在线上看。本地 file:// 已能验证所有 SVG 行为。
- ❌ 不要为了避免撞图就把新元素挤到边角字号缩到 9pt 以下——挪坐标或加 step-end，不要牺牲可读性。

## 不要做的事

- ❌ 引入 npm / 任何构建步骤
- ❌ 把单页教学（如 beginner/index.html）拆成多文件（单文件 + URL hash 是核心 UX）
- ❌ 修改主题色板（所有专题视觉统一在 cream + cherry red，改色板影响巨大）
- ❌ 给概念性 step 强加 Rust 代码（占位文字已显式去除，没 code 就让卡片不显示更干净）
- ❌ 跨专题共享 JS / CSS 文件（每个专题必须 self-contained）
