# SVG 撞图 / 覆盖问题排查

> **何时翻这篇**：在浏览器里跑某个专题，发现 SVG 元素相互叠在一起 / 上一 step 的内容没消失 / 清屏后还有残留 / 跨 step 标签语义不符。
> **不适用**：JS 控制器本身坏了（goTo 不响应、键盘失效、URL hash 没同步）—— 那是 controller bug，直接读 `e2ee/beginner/index.html` 末段 JS 对照。

## 背景假设

每个专题的渐进教程是单 HTML，SVG 用 `data-step="N"` + 可选 `data-step-end="N"` 属性控制元素累积演进。`goTo(n)` 函数遍历所有 `[data-step]` 元素，按 `n >= start && n <= end` 切 `.visible` class。**不写 step-end 默认 end=999**，意味着该元素一旦出现就永远累积到末步。

99% 的撞图问题源于：**该消失的元素没加 step-end**，或新加元素的坐标跟已存在元素的 bounding box 相交。

## 标准排查工作流（7 步）

1. **跑 mcp，逐步截图**：用 `chrome-devtools-mcp` 打开 `file:///.../<topic>/<sub>/index.html#step-N`，对**每个章节的边界 step**（章节末步 + 下一章首步）+ **每个引入大块新元素的 step** 截图。不要只看 Step 1 和 Step N，重点看"刚刚出现新元素"和"刚刚清屏"的步。同时 `list_console_messages(types=['error','warn'])` 检查 JS 是否报错。

2. **对照视觉问题分类**：
   - **文字叠在一起** → 上一 step 的文字没 `data-step-end`，新 step 在同 y 区域加了新文字
   - **框/形状叠在一起** → 上一 step 的图形元素没 step-end，新 step 在重叠 x/y 区域加新框
   - **旧概念应该消失但还在** → "替换型"展示没用 step-end 做交接（例：Step 5 的新 active/inactive 摞框替代 Step 3 的 SessionRecord 框，Step 3 必须 `data-step-end="4"`）
   - **标签描述跟当前 step 语义不符** → 上一 step 给某元素配的副标在新 step 还显示，但语义已变（例：Step 11 的 Server 副标 "key distribution" 在 Step 12 stale 场景里就变成误导）

3. **定位代码**：
   ```bash
   rg -n 'data-step="N"' <file>.html
   ```
   列出该 step 所有元素，再 `Read` 看这些 group 的 `<rect> <text> <line>` 坐标。

4. **算重叠**：拿出涉嫌撞图的两个元素的 `x / y / width / height`，心算 bounding box 是否相交。常见场景：viewBox 0-1100 × 0-520，新元素在 y=380-500，旧元素在 y=370-420 → 必撞。

5. **选修复策略**（按优先级从高到低）：
   - **加 `data-step-end="N"` 限定可见窗口** — 适用于 hint 文字 / 警告 callout / 替换型展示。最简单，优先选。
   - **拆分原 step group**：把"持续元素"（如 actor 圆 / Server 椭圆）和"step-only 元素"（如 OutEnvelope 详情 box）拆成两个 group，前者裸 `data-step="N"`，后者加 `data-step-end="N"`。当一个 group 里既有要保留的也有要消失的元素时用。
   - **重新选坐标**：新元素挪到空白区。优先挪而不是缩，缩文字会丢可读性。
   - **改副标使其语义跨 step 通用**：例 "key distribution" → "目录服务"，覆盖 prekey + device list 两种用途。当一个持续元素的副标在后续 step 变得误导时用。

6. **用 `evaluate_script` 验证假设**：截图模糊时，跑一段 JS 拿到精确的 bounding box / class / data-* 属性，比肉眼准。例：
   ```js
   () => Array.from(document.querySelectorAll('.chapter-svg.active rect'))
     .map(r => ({ x: r.getAttribute('x'), y: r.getAttribute('y'),
                  w: r.getAttribute('width'), h: r.getAttribute('height'),
                  stepEnd: r.closest('[data-step]')?.dataset.stepEnd }))
   ```

7. **修完 reload 再截图**：`navigate_page(type='reload', ignoreCache=true)`，**不要相信缓存**。截图前后对比，确认问题消失且没引入新问题。

## 加新 step 之前的预防 checklist

- 上一 step 在同 y 区域 (±30px) 是否有不该保留的元素？没 step-end 就加 step-end。
- 新加的"概念性面板/算法 box"是否撞了已有的 Server / 表格 / 长条？先在脑子里画 bounding box 再写坐标。
- 新加的"替换型"元素（旧概念被新表达替代）—— 旧元素的 group 必须加 `data-step-end="<新 step - 1>"`。
- 顶部 hint 文字必须 `data-step-end="N"`（每步只活一步），不能裸 `data-step="N"`。

## 不要这样排查

- ❌ **不要只看代码就断定"应该没问题"**。SVG 累积演进的撞图必须实跑才能发现，bbox 算错的概率比想象中高。
- ❌ **不要用 sleep 等 GitHub Pages 部署后再在线上看**。本地 `file://` 已能验证所有 SVG 行为，部署只关心 host 路径。
- ❌ **不要为了避免撞图就把新元素挤到边角字号缩到 9pt 以下**——挪坐标或加 step-end，不要牺牲可读性。

## 真实案例（修过的）

可在 git log 里搜 "Fix" 看具体提交：
- ZKP Step 9 (`35a7ad1` 后续) — Step 8 Knowledge Soundness 紫框 + Step 9 verify accept 框右下重叠 → 给 Step 6 nonce warning + Step 8 Knowledge Soundness 加 `step-end="8"`
- ZKP Step 17 (同上) — Step 14-16 应用展示元素累积到 Step 17 思维导图，盖住中央 ZKP 节点 → Step 15 / 16 加 `step-end="16"`
- Noise Step 11 (同上) — `s_A` 框 y=200 跟 Alice actor 圆 (cy=240, r=36) 重叠 → s_A 移到 y=400 镜像 Bob 的 s_B（坐标重选）
- Sesame Step 5 (`ff817fb`) — Step 3 SessionRecord 框跟 Step 5 active/inactive 摞框完全叠（替换型展示）→ Step 3 加 `step-end="4"`
- Sesame Step 8 (同上) — Step 7 OutEnvelope 紫框 (x=380-660, y=370-420) 被 Step 8 fan-out cherry 框 (x=60-620, y=380-500) 部分遮 → 把 OutEnvelope 拆出独立 group + `step-end="7"`（group 拆分策略）
- Sesame Step 12 (同上) — Step 11 Server 副标 "key distribution" 在 Step 12 stale / Step 13 设备移除场景里语义不符 → 改为 "目录服务"（副标语义跨 step 通用化）
- Signal DR Step 24 (`9a0d239`) — 时间轴 line y=280 被 FS/受损/PCS rect (y=240-280) 完全遮，加上一个多余的字面 `→` 字符 → line 下移到 y=300 + 删多余字符（坐标重选）
