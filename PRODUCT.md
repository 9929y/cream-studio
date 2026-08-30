# Product

## Register

product

## Users

Yanice（产品设计师）本人。使用场景：桌面浏览器打开单文件工具，在七种生成器之间切换，调参数出图，导出 PNG / SVG / GIF 用于作品集、社交分享、或作为其他设计稿的素材。接麦克风时可以让画面跟着现场声音走，用于现场演示和录屏。

## Product Purpose

Cream Studio 是一个单文件 HTML 生成艺术工作台。它不是一个生成器，而是**一个仪器盘外壳装七个生成器**：DOTS（点阵）· ARCS（弧形）· ORBS（圆环）· FIELD（等高线地形）· AURA（扩散渐变）· SLICE（图像切碎）· FLOW（粒子流场）。七者共用同一套 seeded RNG、四档运动、三频段音频响应和导出管线，所以在它们之间切换的成本接近零。

成功标准：同一个 seed 永远得到同一张图；拖滑杆即时可见；所有预设在 60fps 预算内；双击 `index.html` 就能跑，不联网也能导出 GIF。

## Brand Personality

三个词：中性、精密、退后。SKILL.md 的原话是 "UI is neutral, artwork is loud" —— 界面不许有颜色，颜色只允许出现在画布里。**这一条到 2026-08-27 的 visionOS 改版为止始终成立**：界面从暖灰换成了暗色玻璃，但仍然是零彩色，唯一的色相来自背景图和画布。

参照物在 2026-08-27 由「实验室仪器面板」改为 **Apple visionOS**：一扇浮在环境之上的玻璃窗。见偏离第 10 条。

## Anti-references

- UI 里出现任何强调色（SKILL.md 硬规则 2 的这一半**始终无条件有效**。两轮改版加了圆角、渐变、玻璃，但一次都没有加彩色）
- 与操作无关的纯装饰、投影堆叠、bounce/elastic 缓动
- 用 `Math.random()` 决定布局（构图必须能靠 seed 复现）
- 把七个模式做成七个文件（那样共用系统会立刻分叉）

## Design Principles

1. **画布是主角，面板是仪器** —— 面板只负责操作反馈，一切视觉重量留给 canvas
2. **确定性优先** —— 同 seed 同构图。`Math.random()` 只出现在 reseed、随机配色、音频逐帧抖动三处
3. **运动只推进相位** —— 动画绝不重新随机布局，缓存的 layout 上叠旋转 / 相位 / 呼吸
4. **一个模式接口** —— `MODES[name] = { groups, defaults, looks, build, draw, toSVG?, count }`，外壳只认这个契约，加第八个模式不需要动外壳
5. **导出即成品** —— PNG 是当前帧原尺寸，SVG 是真几何而非嵌图，GIF 是完整一轮相位循环

## Deviations from SKILL.md

规范原件在 `02_agent_memory_and_skills/generative-tool-bycoraldesign/视觉生成器SKILL/SKILL.md`，它仍是参数默认值与架构的权威。**不要去改那份文件**（`redskill upgrade` 会整份覆盖，且改它等于伪造上游意图）。以下偏离逐条记在这里，后续会话不要「修正」回去。

### 1. 一个外壳装七个模式（Yanice 2026-08-27 明确要求）

SKILL.md 的语气是「一个工具一个单文件」。Yanice 的要求是「实现 skill 里所有的效果」，也就是把 SKILL.md 第 10 行列出的作者既有系列（FIELD / AURA / SLICE / dot matrix / arc circle / orb rings）加 frontmatter 提到的 particle systems 全部做出来。做成七个文件会让共用系统立刻分叉，所以做成一个外壳 + 七个模式。SKILL.md 的 UI 骨架本来就写了 "right side holds mode tabs"，这是规范内的用法。

### 2. 交付路径（本机环境限制，非审美选择）

硬规则 6 要求交付到 `/mnt/user-data/outputs/` 并用 `present_files` 呈现。那是 Claude.ai 分析工具的环境，本机不存在。按库规则落在 `01_projects/vibecoding/cream-studio/`。

### 3. GIF 用内联编码器，不引 gif.js

硬规则 1 允许「按需从 cdnjs 引 gif.js」。这里改成移植 `letter-shape-generator/index.html` 里那份零依赖手写 GIF89a 编码器（首帧 RGB555 桶采样取 256 色全局调色板 + 32768 项最近邻查表 + 带位宽增长和字典重置的标准 LZW）。结果是工具**完全离线**，比引 CDN 更严格地满足硬规则 1 的意图，不是放宽。

### 4. 音频区的配色（SKILL.md 自相矛盾，按硬规则裁决）

SKILL.md 的音频小节写「mic button + green live dot in topbar, three 3px VU bars (blue/green/tan)」，这与硬规则 2「Never use accent colors in the UI — all color lives on the canvas」**直接冲突**。裁决：硬规则优先。live 点和三条 VU 条全部用 `--text` 暖灰 token，三条靠位置和不透明度（1 / .66 / .4）区分，不用颜色区分。三频段的映射关系（Bass→尺寸 / Mid→速度 / Hi→抖动）与 20-200 / 200-2500 / 2500-8000 Hz 的分段**原样保留**。

### 5. 面板顶部增加固定常用区（Yanice 2026-08-27 选定的 category 结构）

SKILL.md 只规定「footer pinned at bottom with Randomize + primary Export」。这里在面板顶部另加一个固定区放 Looks 预设 + Seed —— 来自 `letter-shape-generator` 的做法。Yanice 在开工前明确选了「造字机 + Fluid 结合式」的分类结构，即：顶部固定 Looks + Seed（造字机式）、下面可折叠参数分组、色彩用 Fluid 式命名主题（Cream / Cocoa / Mist / Pistachio / Berry / Mono）。底部 footer 仍按 SKILL.md 保留 Randomize + Export。

### 6. FLOW 也给 SVG 导出

SKILL.md 说 SVG 只给「vector tools」。流线本来就是矢量数据，导出成 `<path>` 对绘图仪 / 后期编辑都有用，所以开了。真正的栅格模式 AURA 和 SLICE 的 SVG 按钮是禁用的，并在 title 里写明原因。

### 7. impeccable 的「奶油底色禁令」在这里不适用

impeccable 技能把「暖中性近白底（OKLCH L 0.84-0.97, C < 0.06, hue 40-100）」列为 2026 年饱和的 AI 默认色而禁用。但 `--bg:#f0eeeb` 是 SKILL.md 明文规定的 token，且 Yanice 在开工前明确选了「SKILL 原教旨暖灰盘」。**以 SKILL.md 为准。** 下一个会话不要以「反 AI 味」为由改配色。

## 包内缺失的 `references/code-patterns.md` —— 这些 pattern 是怎么实现的

SKILL.md 三处引用 `references/code-patterns.md` 并写明 "read it before writing code"，但 RedSkill 只发了 SKILL.md。以下按 SKILL.md 的文字描述自行实现，位置记在这里备查：

| SKILL.md 里的名字 | 本项目的实现 |
|---|---|
| `sl(id, key, suffix, parse)` 滑杆绑定器 | 泛化成**声明式 param spec**：每个模式在 `groups` 里声明 `{k, label, min, max, step, struct}`，由 `mkRange()` 统一建 DOM、绑事件、同步读数、按 `struct` 决定是否 `invalidate()` |
| `bindHex(inputId, swatchId, key)` | `mkHex()` —— swatch 按钮 + 文本框 + 隐藏的 `<input type=color>`，`/^#[0-9a-fA-F]{6}$/` 校验，blur 时回滚非法值 |
| 加权色表 | `renderColorList()` —— swatch + hex + 权重 + 实时 %，>1 条时可删，每条一个 `◑` 按钮切单色/渐变并露出第二个 hex |
| 布局缓存 | 模块级 `cached` + `invalidate()`。`drawFrame()` 里 `if(!cached) cached = M.build(...)`。结构性参数标 `struct:true` 自动失效缓存，样式性参数只重绘 |
| 音频完整实现 | `AU` 对象 + `bandOf(lo,hi)` 按 `sampleRate/fftSize` 换算 bin 区间 + `sampleAudio()` 做 EMA 平滑，输出 `AU.size` / `AU.spinRate` / `AU.jit` 三个乘子，各模式通过 `bre()` 和 `jit()` 两个共用函数消费 |

## Accessibility & Inclusion

- 正文对比度 ≥ 4.5:1。因为界面是**玻璃叠在背景图上**，校验按「背景图最亮处 + 全部图层叠完」的最坏情况做，逐层的数字见偏离第 10 条的表。最紧的一处是 topbar 与画布角标上的 `--ink-3`，4.85:1
- 弹层与导出浮层单独验过：它们会压在接近纯白的奶油画布上，按底色 240 验算白字 12.83、`--ink-3` 6.27
- `prefers-reduced-motion`：UI 过渡压平到 1ms。画布动画是内容本体，默认停在 Stop 档，由用户显式开启
- 键盘可达：滑杆原生可键控（visionOS 无滑块，所以 focus 时会浮出白色圆点作为位置指示）、折叠区是真 `<button>` 带 `aria-expanded`、种子框支持 `↑` `↓` 步进、`Space` 播放/暂停、`Esc` 关弹层、`:focus-visible` 焦点环
- 模式切换是 `<nav>` 里的按钮组，VU 条有 `role="img"` + `aria-label`，GIF 进度条是 `role="status" aria-live="polite"`
- `font-variant-numeric: tabular-nums` 全局开启：SF Pro 默认不是等宽数字，拖滑杆时读数会左右跳

### 9. 面板改版：胶囊化 · 渐变表面 · 三层收纳（Yanice 2026-08-27 明确要求）

Yanice 的原话：「所有的 panel 里面的信息太多了，我在用户点的时候不知道点哪个，我想把这个东西 minimize 一些……我希望它用的是胶囊，不要用这种非常方块的这种格式，就是要有 corner radius，然后颜色不要这么死板，有点渐变的背景啊，然后其他的颜色稍微做高级一点。」

这一条**成建制地替换了 SKILL.md 的视觉与信息架构**，但**没有碰硬规则 2**：整套 UI 仍然零彩色，颜色依然只存在于画布。三个岔路口都由 Yanice 当场选定：配色走「精致中性 + 微渐变（无强调色）」、信息走「三层收纳」、面板宽度走「可拖拽 280–420」。

**改了什么：**

1. **token 表整体替换**（作废本文件第 8 条里的 `--text2/--text3/--border2` 命名，但那一条要求的 ≥4.5:1 继续有效，新值见上面的 Accessibility 表）。新增的是**表面**概念：`--panel-top/--panel-bot`（面板竖向渐变）、`--stage-c/--stage-e`（舞台径向渐变）、`--raise`（浮起面）、`--sunk`（凹槽轨道）。激活态从纯黑换成深咖啡 `--active #2b2622`。
2. **全部控件改胶囊。** 分段控件是「凹槽轨道 + 反色胶囊滑块」，按钮、预设、色块、输入框、麦克风全部 `border-radius:999px`。圆角面（弹层、进度浮层）用 14px —— 卡片类元素不超过 16px，避免过度圆角。
3. **渐变背景。** 面板 `#f5f3f0 → #e9e6e1` 竖向；舞台是以画布上方为焦点的径向渐变 `#ebe8e4 → #dcd8d3`，画布因此有轻微的「被打光」感。
4. **三层收纳。首屏控件从 25 个降到 9 个。**
   - 固定头部：预设 · 种子 · 色板（一排色块）
   - 第一层：当前模式的 **4 个主参数** + 运动档
   - 第二层「更多参数（n）」：该模式其余参数 + 方向/层速差/呼吸
   - 第三层「设置」：画布与音频（是配置，不是创作）
5. **加权色表收成一排色块。** 原来是 5 行 × 7 个控件。现在头部只有一排圆色块 + 主题胶囊 + `+`；点任一色块弹出该条的色值 / 单色渐变 / 权重 / 删除。弹层是 `position:fixed`，不会被面板的滚动容器裁掉。手动改过颜色后主题胶囊显示「自定」。
6. **去掉三重冗余的反馈。** topbar 原来的 `MODE / SIZE / COUNT` 与画布四角标签、statusbar 三处重复。现在：**画布四角**是作品元数据（SEED / COUNT / MOTION / SIZE），**statusbar 变成瞬时通道**（音频三频段 → 导出结果 → 否则是一条随模式变化的提示）。topbar 只剩品牌、模式胶囊、麦克风。
7. **面板可拖拽改宽** 280–420，默认 340，宽度记进 localStorage，双击把手复位。
8. **滑杆有了已填充段**，用 `--ink` 填充、`--sunk` 做未填充轨道。这是纯反馈增强，零成本。

**踩到并修掉的坑：** 拖拽把手的 `pointerdown` 里调 `preventDefault()` 会连带吞掉浏览器补发的 `click`/`dblclick`，导致双击复位失效。改成用 CSS `touch-action:none` 处理触屏、不再 `preventDefault`。

### 10. 改用 Apple visionOS 设计系统（Yanice 2026-08-27 明确要求）

Yanice 给了 Figma 链接 `Apple visionOS UI Kit V1.0 (Community)`，fileKey `BpD9qad9rV0yrLjkWeoR64`，要求「use this design system to redesign current UI」。三个岔路都由她当场选定：**完整 visionOS（舞台一起变暗，背景图她提供）· 全部换 SF Pro**。

**token 全部是从 Figma 文件里读出来的，不是凭 visionOS 的印象写的。** 读取节点：`motion setting` 4:4338 · `window top tab` 4:5524 · `video bottom bar` 4:5497。

| | 从 Figma 读到的值 | 用在哪 |
|---|---|---|
| 窗口玻璃 | `backdrop-blur(50px)` + `rgba(0,0,0,.14)` + `inset 0 ±1px 1px rgba(255,255,255,.15)` + `0 8px 6px rgba(0,0,0,.05)` | `.app` 整扇窗 |
| 侧栏玻璃 | `rgba(0,0,0,.10)` + `inset 0 0 1px rgba(255,255,255,.15)` | `.panel` |
| 凹槽控件行 | `rgba(0,0,0,.14)`，r14，h58，padding 20 | 每个滑杆是一张凹槽卡片 |
| 胶囊按钮 | h36 r20 `rgba(255,255,255,.30)`；容器 h48 r100 `blur(40px)` | mode tabs · 分段控件 · Looks · 按钮 |
| **滑杆** | 轨道 h14 r22 `rgba(255,255,255,.10)` + `inset 0 -1px 2px rgba(255,255,255,.40)`；填充 `rgba(255,255,255,.60)`；**没有滑块** | 全部参数滑杆 |
| 输入框 | r24 h36 `rgba(0,0,0,.14)` + `inset 0 -1px 2px rgba(0,0,0,.04), inset 0 2px 2px rgba(0,0,0,.14)` | 种子框 |
| 圆角尺度 | 10 · 12 · 14 · 18 · 20 · 22 · 24 · 54 · 100 | 全局 |
| 字号 | 12 / 14 / 16 / 24，SF Pro Display Regular / Medium | 全局 |
| 文字 | `#fff` · `rgba(255,255,255,.74)` · `rgba(255,255,255,.64)` | 三级 |

**背景图**：Yanice 提供的 `~/Desktop/Cover.jpg`（7680×4320，1.4 MB），原件归档在 `assets/background-source.jpg`。`index.html` 内联的是压到 900px 宽、quality 62 的版本，**12.9 KB**（base64 后 16.8 KB）—— 原图是虚化渐变，放大回全屏看不出差别，所以单文件与离线可用都保住了。细节见 `assets/SOURCE.md`。

#### 对比度是解出来的，不是估出来的

visionOS 的玻璃能只用 `rgba(0,0,0,.14)` 就保证可读，是因为**系统会把窗口后面的实景压暗**。这里照做：`--scrim: rgba(6,8,12,.45)` 是按背景图**实测最亮处 153** 反解出来的（实测亮度范围 107–153，中位 112）。这样才能用回规范里的 `.14` 而不是随便加浓玻璃。

实际图层叠加与最坏情况对比度：

| 文字所在层 | 叠加后灰阶 | 白字 | `--ink-2` .74 | `--ink-3` .64 |
|---|---|---|---|---|
| topbar / 画布角标（scrim + 窗口玻璃） | 72.4 | 9.09 | 5.87 | 4.85 |
| 面板底（+ 侧栏玻璃） | 65.1 | 10.19 | 6.45 | 5.29 |
| 参数行卡片（+ 凹槽） | 56.0 | 11.72 | 7.25 | 5.87 |

⚠️ **换背景图必须重算 scrim。** 换一张更亮的图，`--ink-3` 会先跌破 4.5:1。

#### 四处 visionOS 到 2D 网页翻译不过来的地方

1. **弹层与导出进度浮层用了 `rgba(16,18,23,.86)` 而不是规范的 `.14`。** 它们会压在画布上，而画布可能是接近纯白的奶油色（240）——那不是被压暗的实景。按 240 验算：白字 12.83、`--ink-3` 6.27，仍然是玻璃（有模糊、透得过内容），但底够实。
2. **滑杆的滑块只在 hover / focus 时出现。** visionOS 的滑杆真的没有滑块，因为它靠眼动定位。鼠标和键盘需要抓取提示，所以默认无滑块（保住那个标志性外观），指针悬停或键盘聚焦时才浮出一个白色圆点。
3. **画布四角的十字准星标记删掉了。** 那是仪器/印刷的语汇，和玻璃拟态打架。四个角标文字保留（它们是作品元数据，有用）。
4. **窗口底部的抓握条只在窄屏出现。** visionOS 里它是用来拖动窗口的；网页里没有窗口可拖，留着就是个死控件。改成移动端底部抽屉的握把，那里它是真的有功能。

#### 一条实现上的注意

`.mtabs` 故意**不带** `backdrop-filter`。`.app` 已经把环境模糊过了，在它内部再套一层只是模糊已经模糊过的像素，视觉上零差别，却多一个合成层。同理，弹层 `.pop` 挂在 `document.body` 上而不是 `.app` 里面 —— 除了避免被面板的滚动容器裁掉，也顺带避开嵌套滤镜。全文件只剩两处 `backdrop-filter`：`.app` 和 `.pop` / `.prog`。

#### 硬规则现状

- 硬规则 1（单文件）**更严格了**：SF Pro 走 `-apple-system` 系统字，Google Fonts 那条 `<link>` 已删除，现在**零外部请求**
- 硬规则 2 的「UI 不许有强调色」**仍然成立**，零彩色；「只用暖灰 token 表」那一半已被本条整体替换
- 硬规则 3（seeded 可复现）· 4（中文滑杆 / 英文分组）· 5（data URI + 可见兜底链接）**全部照旧**

### 11. 导出不再自动触发下载（Yanice 2026-08-27 报的 bug）

Yanice 的原话：「我每次离开他都会自动提示我去 save 一些图片/gif 这是不合理的吧，得用户主动选才会有。」

原因是我在 `offer()` 里调了 `a.click()`。导出几次之后 Chrome 就会弹「此站点想要下载多个文件」的授权框，看起来像工具在追着用户存文件。**这是我的错误决定，不是 SKILL.md 要求的** —— 硬规则 5 只要求「data URI + 留一条可见的兜底链接」，从来没要求自动点它。

改成两态按钮：

1. `Export PNG` → 只**生成**，不碰下载管理器
2. 按钮变成 `↓ 下载 cream-dots-42.png`，下面出现一行「没反应的话右键这里另存为」（硬规则 5 的兜底）
3. 再点一次才真的保存 —— 这一次是用户主动选的
4. 期间改任何参数、换模式、换导出格式，暂存作废，按钮变回 `Export PNG`

实测：跑完 7 个模式 × 35 套预设的全部切换，**意外触发下载 0 次**。

### 12. 浅色主题（Yanice 2026-08-27 要求「看一下 light 模式长什么样」）

#### 先说一个从 Figma 里读出来、和直觉相反的事实

**visionOS UI Kit 里没有「深色文字的浅色模式」。** 它那些 `light/dark` 变体切换的只是**玻璃的色调**，文字在两个变体里**都是白的**：

| | 深色变体 (4:5497) | 浅色变体 (4:5470) |
|---|---|---|
| 窗口玻璃 | `rgba(0,0,0,.14)` | `rgba(255,255,255,.14)` |
| 滑杆轨道 | `rgba(255,255,255,.10)` | `rgba(0,0,0,.20)` |
| 滑杆填充 | `rgba(255,255,255,.60)` | `rgba(255,255,255,.60)` |
| 参数行凹槽 | `rgba(0,0,0,.14)` | `rgba(0,0,0,.14)` |
| **文字** | **白** | **白** |

因为两个变体都假设自己压在**暗环境**上 —— 「浅色」指的是玻璃比环境亮，不是界面变成浅底深字。把浅色变体渲染在白色画板上，字几乎看不见（Figma 的官方预览图就是这样）。

**所以真正意义上的浅色模式是在这套系统之上的推演，不是照抄。** 全部值里只有两个是有出处的：浅色窗口玻璃 `rgba(255,255,255,.14)`（节点 4:5470）、浅色滑杆轨道 `rgba(0,0,0,.20)`（节点 4:5492）。其余都是解出来的。

#### 三处必须反过来的地方

1. **压暗 → 提亮，但主要靠玻璃。** 暗色模式是把环境压暗（scrim .45）好让 `.14` 的薄玻璃可读。浅色模式如果照样只靠 scrim 提亮，背景图会被洗白、失去意义。改成：环境只轻提亮（白 scrim `.18`），窗口换成**厚白霜玻璃** `rgba(255,255,255,.62)`。背景图仍然透得过来，窗口足够亮。
2. **凹槽 → 浮起。** 参数行在暗色里是 `rgba(0,0,0,.14)` 的凹槽。浅色里沿用会把表面压到 184，`--ink-3` 掉到 **3.57:1，不达标**。改成白色浮起面 `rgba(255,255,255,.55)`（表面 240）—— 这也更符合浅色界面的常规：卡片是浮起的，不是凹陷的。
3. **最坏情况换了一端。** 暗色模式按背景图**最亮处 153** 解（白字最难读）；浅色模式按**最暗处 107** 解（深字最难读）。

#### 浅色模式实测对比度（按最暗处 107 逐层叠完）

环境 134 → 窗口 209 → 面板 223 → 参数卡片 240

| token | 窗口 | 面板 | 参数卡片 |
|---|---|---|---|
| `--ink` `#1d1d1f` | 11.03 | 12.62 | 14.86 |
| `--ink-2` `#46464b` | 6.17 | 7.06 | 8.32 |
| `--ink-3` `#58585f` | 4.65 | 5.32 | 6.27 |

主按钮 `#1d1d1f` + 白字 16.86 · 激活胶囊 `rgba(0,0,0,.82)` + 白字 14.10。全部达标。

#### 顺带修的一个真隐患

做浅色的过程中发现 `.btn` 写的是 `background:var(--glass-btn); color:var(--ink)`。暗色下没问题（浅胶囊 + 白字），但 `--glass-btn` 在浅色里是**深色**激活胶囊，配 `--ink` 的深色文字就是**深底深字**。拆成独立的 `--btn-bg` / `--btn-ink`，`--glass-btn` 只保留「激活态胶囊」这一个语义。

同时把 CSS 里三十多处硬编码的 `rgba(255,255,255,...)` 收成语义 token（`--hairline` `--hover` `--btn-hover` `--sw-ring` `--pop-bg` `--canvas-shadow` `--base` `--focus` 等），否则浅色主题根本无法只靠换 token 实现。

#### 怎么切

设置 → 外观：**跟随系统 / 浅色 / 暗色**，默认跟随系统，选过之后记进 localStorage。CSS 用
`@media (prefers-color-scheme:light){ :root:not([data-theme="dark"]) }` 加 `:root[data-theme="light"]`
两条，所以「跟随系统」和「显式选择」都成立。

#### 诚实的代价

**浅色模式下画布不再「浮」起来。** 奶油色的作品压在同样浅的窗口上，那种「暗房里挂着一幅画」的效果没有了 —— 这是浅色模式固有的，不是没调好。加深了画布投影缓解，但改变不了根本。暗色仍然是这个工具更好看的那一面。

#### 用到的设计技能

`redesign-skill`（2026-08-27 起激活）与 `taste-skill`。taste-skill 的 §13 明确把密集型产品 UI 列为适用范围之外（它面向落地页/作品集），所以只用了真正管得着的几条：Dark Mode Protocol §8（双模式、跟随系统、不用纯黑纯白、两个模式层级对等）、Theme Lock §4.11、形状与配色一致性锁、以及 pre-flight 里的 Button Contrast Check —— 上面那个 `.btn` 深底深字就是被这一条抓出来的。

### 13. English UI, icon mic, four-tab scroller, one pale accent (Yanice 2026-08-27)

Four requests in one round.

#### 13a. The whole UI is English now

**This overrides SKILL.md hard rule 4** ("Chinese labels for parameters, English for section headers"). Yanice: 「整体用英文 别出现中文」. Every visible string, `aria-label`, `title` and status message was translated; the file now contains zero CJK characters. Section headings were already English, so only the parameter labels, segment options, popover copy and hints changed. The English labels are wider than the Chinese ones, so the pinned-header label column went from 36px to 52px and the swatches from 24px to 22px to keep the Palette row on one line.

#### 13b. Mic is an icon and moved left

The `MIC` text pill became a 36px circular icon button and moved from the far right to just left of the tab strip, so the topbar now reads identity → input status → navigation.

**The glyph is the kit's own `mic.fill`**, exported from Figma node `2:6715` and inlined as a path with `fill="currentColor"` so it themes. Not hand-drawn: `figma-design-to-code` is explicit that icons must come from the exported asset, and taste-skill §3.C bans hand-rolled SVG paths. Inlining rather than linking also keeps the asset alive past Figma's 7-day URL expiry and preserves zero external requests.

#### 13c. Tab strip shows four, scrolls for the rest

`.mtabs` is now a snap-scroller sized to `4 × 76px + 4 × 4px + 16px`. The 16px tail means **the fifth tab shows as a sliver**, which is the scroll affordance — no arrows, no dots, no `Scroll →` label (taste-skill bans scroll cues). Three visible at ≤860px. The pill track moved to a `.mtabs-wrap` so the scroller's overflow does not clip the rounded ends.

#### 13d. One pale accent replaces every black active state

Yanice: 「下方 controller 颜色不要用黑色 用一个 pale color」. In light mode the active pills and the Export button were near-black `#1d1d1f`, which read as two harsh slabs in the footer.

**The accent is sampled from the background image, not invented.** The image's lower-left teal is `#6c8ca0`; lifted 62% toward white it gives **`--accent: #c7d3db`**, 11.03:1 against `--on-accent #1d1d1f`. It now carries *every* active and primary state in **both** themes (mode tabs, Looks, segments, mic-on, Export), which also satisfies taste-skill §4.2's Color Consistency Lock — one accent, used identically everywhere, rather than a pale footer above a black topbar.

**The problem this surfaced, and the fix.** A pale fill on a light panel reaches only **1.14:1** — WCAG 1.4.11 wants ≥3:1 for a UI state boundary, and *no* genuinely pale colour can hit that against a 223 surface. So in light mode the fill cannot carry the state alone. The boundary does instead: `--active-ring: #4e697a` (the same image teal, deepened) as a 1.5px inset ring, measuring 4.35:1 against the panel and 3.18:1 against the segment track. Dark mode needs no ring — the pale fill is already 6.69:1 against the glass — so `--active-ring` is `transparent` there.

| | dark | light |
|---|---|---|
| accent fill vs its surface | 6.69 | 1.14 |
| ring vs surface | not needed | 4.35 |
| `--on-accent` on the fill | 11.03 | 11.03 |

#### Hard-rule status after this round

- Rule 1 (single file, no external requests): **still holds** — the icon is inlined
- Rule 2's "no accent colour in the UI": **now broken.** Two rounds of restyling kept it; this one deliberately introduces one accent, at Yanice's request. It is a single desaturated colour taken from the project's own background image, used only for active and primary states
- Rule 3 (seeded, reproducible): still holds
- Rule 4 (Chinese parameter labels): **now broken**, see 13a
- Rule 5 (data URI + visible fallback link): still holds

### 14. Single-panel layout, light-only, curated defaults (Yanice 2026-08-27)

Seven changes in one round. Several of them undo earlier deviations, so this section is the current truth where it conflicts with 10-13.

#### 14a. Layout: one panel right, bare preview left

The floating visionOS *window* (`.app`, radius 54, glass, holding topbar + stage + panel) is gone. The page is now two columns directly on the background image:

- **left** — the preview alone. No glass box under it (Yanice: 「左侧有个 preview（底下没有玻璃 box）」). The canvas sits on the environment with only its own shadow.
- **right** — one floating glass panel, radius 32, holding *everything*: brand, mic, Style, Presets, Palette, Fill, parameters, Motion, More, Settings, Randomize/Export and the status line.

The separate topbar and statusbar zones are gone; the status line moved to the bottom of the panel.

**Contrast had to be re-solved.** With no window glass in the stack, the panel now sits straight on the scrimmed image and carries all the type by itself. Solved against the image's darkest point (107): `--scrim: rgba(255,255,255,.12)` and `--panel: rgba(255,255,255,.72)` give a panel at 218.5 and cards at 238.6, where `--ink` 12.12 / `--ink-2` 6.79 / `--ink-3` 6.16. A lighter panel (`.62`) put `--ink-3` under 4.5 and was rejected.

#### 14b. Light only

The appearance control and the whole dark token set are **deleted** (Yanice: 「appearance 删掉 只有浅色模式」). This reverses deviation 12's dual-theme work; the light values are now the only `:root`. `--active-ring` is no longer conditional since there is only one theme to ring.

#### 14c. Style and Presets are one-row scrollers

Both are horizontal snap-scrollers inside a pill track. Style shows four with a sliver of the fifth as the scroll cue (three at ≤860px); Presets is a single row that scrolls (Yanice: 「Preset 就一行 左右滑动出现」).

#### 14d. Seed moved into Settings

It is a reproducibility control, not a creative one, so it no longer occupies the pinned header. `setSeed()` had to become tolerant of `#seedIn` being absent, since the field only exists while Settings is expanded.

#### 14e. Presets no longer carry a palette

`theme:` was stripped from all 35 presets, and `applyLook()` no longer touches `S.colors` (Yanice: 「每一个 preset 都用同一套 Color palette」).

**Consequence that had to be fixed:** presets were tuned against their own palettes, and the shared default (Cream) had its top two tones within 6 grey levels of the canvas, so Halftone in particular vanished. The Cream ramp was widened from `#fdf3e3…#ab7647` (contrast range 1.05-3.35 against the canvas) to `#f7e7cb…#7b5530` (1.05-5.70).

#### 14f. Every mode opens on a curated preset

`baseSeed` alone was the wrong lever: a contact sheet of 12 seeds on DOTS showed almost no meaningful variation, because the seed only shifts per-dot jitter, not composition. What actually makes an opening view strong is the parameter set.

So the **first** visit to a mode applies `MODES[name].defaultLook || 0` (`seenMode` guards it), and returning to a mode keeps whatever you edited. Default seed is now 7.

#### 14g. DOTS had no motion, and the fix had two layers

Reported as 「dot 现在没有动效」. Two separate causes:

1. DOTS defaulted to `warp:"none"`, so the Motion segment drove nothing but a 0.06 breathe. Default is now `warp:"sine", warpAmt:20, warpFreq:1.3`.
2. The new opening preset, Halftone, set `warpAmt: 0` — so fixing the default alone would still have left the *opening* view frozen. Halftone is now `warpAmt:30, warpFreq:2.4, breath:0.05`.

Verified empirically rather than by reading the code: `requestAnimationFrame` was temporarily replaced with a synchronous driver (this preview pane pauses rAF), the animation advanced ~30 frames per mode, and the canvas was compared before and after. **All seven modes move.**
