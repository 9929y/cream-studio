# 四个工具的合并决定

> 2026-08-30。读自四个 repo 的当时 HEAD：
> `meshy-studio@3eef749` · `glass-studio@5305606` · `cream-studio@dca97a1` · `letter-shape-generator@7bf5636`。
>
> 这份文件是**决定记录**，不是任务清单。它说明为什么这么切、代价是什么、以及哪些东西
> 后续会话不许「顺手修正」回去。完整版放在 cream-studio，因为 Phase 1 在这里发生；
> 另外三个 repo 各有一份只写「这对本仓库意味着什么」的短版。

---

## 结论：3 + 1，而且那个 3 其实是 2

**Glass Studio 保持独立产品。** 其余三个合并成一个 **Studio**，但**不是三个平级的引擎**——
是**两种图层类型**，在同一个文档里叠：

| | 来源 | 是什么 |
|---|---|---|
| **Field** 层 | meshy-studio | 连续色场。WebGL。是底。 |
| **Marks** 层 | cream-studio + letter-shape-generator | 离散元素。canvas 2D。落在底上。 |
| **Glass**（独立产品） | glass-studio | 透镜。链式。读它下面的再发射出去。 |

两个产品**共用同一套外壳**（同一份 UI kit、同一套六格 IA、同一条后期链、同一个画板模型），
但它们是两个 app，不是一个 app 的两个 tab。

---

## 判据：它需不需要「已经存在的东西」

这是整条切割线，它在每个文件里都成立。

Mesh、Cream、Letter 都是从零合成像素——打开就有画面，不需要任何输入。
Glass 打开的是**素材**：

- `types/glass.ts` 的 `SourceSettings` 带 `mediaUrl` / `mediaName` / `fit` / `zoom` /
  `offsetX/Y` / `rotation` / `playbackRate` / `loop`
- 有 `hooks/useMediaDrop.ts`，别的三个都没有
- 内置程序化渐变存在的理由，它自己写了：
  *"so the studio is never empty and never needs an upload to be useful"*

合成模型上也分得开。Glass 是唯一一个图层构成**链**而不是**栈**的：

```
source ──► glass₁ ──► glass₂ ──► … ──► glassₙ ──► effects ──► out
```

`types/glass.ts` 的原话：*"glass is not a colour laid on top, it is a lens."*

一个透镜和三个生成器不该共用图层语义、共用左栏、共用首次打开的体验。

### 这个切法唯一的代价，以及它的解法

「mesh 渐变 → 上面盖一层竖纹玻璃」这个组合，分家之后要走一趟交接。

但交接本来就已经建好了：Glass 的 plate 收图 / 视频 / GIF。所以：

1. Glass 加一个 `studio-doc` 的 `SourceKind`
2. Studio 的导出里加一个 **Open in Glass**

两边各一次点击。两个产品保持各自诚实，不用为了一个组合把透镜塞进栈里。

---

## 发现：Letter Shape 不是第三个生成器

**它是 Cream 换了一个 sampler。**

两边都是同一条四段管线，只是切口位置不同——而 letter 的切口是对的那一个。

| 阶段 | cream-studio | letter-shape-generator |
|---|---|---|
| **pool**<br>一个 mark *是什么* | 混在 `build(S,p)` 里，决定位置的同时决定 `c` / `r` / `a` / `ph` | `shapes[]` 独立构建（index.html:1183）：`color`（`weightedPick`）、`size`、`opacity`、`trembleFreq`、`tremblePhase`、`breathPhase`、`glyph`、`bricks`。**完全不依赖文字。** |
| **distribution**<br>它*去哪* | 点类模式各返回一个点列表（见下表：只有 DOTS / ARCS / ORBS 是点类） | 只有一个：`assignTargets(text)`（index.html:1218）——采样字形 mask，往**已经存在的** mark 上写 `toX` / `toY` |
| **place(t)**<br>逐帧 | `MODES.dots.place()`，注释原话：<br>*"motion advances warp phase + breathing; it never re-randomises layout"* | tremble / breathe，改同样的字段，做同样的事 |
| **draw**<br>画什么形状 | 形状焊死在 mode 里（DOTS 画圆、ARCS 画弧） | 是个控件：`dot` / `brick` / `ascii` |

**pool 和 distribution 分开**，是 letter 能在多段文字之间炸开重组的原因，也是这次合并的全部价值来源。

### 但只有三个模式能进这个契约（2026-08-30 逐个读过 build 之后修正）

「cream 的七个模式都变成 sampler」是**错的**。七个 `build()` 返回的是**不同的图元**：

| mode | `build()` 返回 | 图元 | 能进点标记契约？ |
|---|---|---|---|
| **DOTS** | `{pts:[{x,y,r,a,c,ph,nd,band}]}` | 点 | **能** |
| **ORBS** | `{orbs:[{R,ring,a,r,al,c,ph}]}` | 极坐标点（`place()` 已经吐 `{x,y,r}`） | **能** |
| **ARCS** | `{arcs:[{R,a,sweep,layer,lw,c,ph}]}` | 极坐标点 + 角度跨度 | **能**（见下） |
| FIELD | marching squares 等高线 | 折线 | 不能 |
| FLOW | `{perm, seeds}` + `trace()` | 折线 | 不能 |
| AURA | `{blobs}` | 径向渐变栅格 | 不能 |
| SLICE | `{src, slices}` | 图像切片栅格 | 不能 |

**能进的那三个，per-mark 身份字段是同一套**：`c`（`pickColor()`）、`ph`（相位）、
一个尺寸（`r` / `lw`）、一个 alpha、一个 ring/layer 索引（喂 `speedDiff`）。
这正好就是 letter 的 `shapes[]`：`color` / `size` / `opacity` / `tremblePhase` / `breathPhase`。

**ARCS 的 `sweep` 是 draw 参数不是 distribution 参数**——弧就是「一个点加一段角度跨度」。
所以 ARCS 不是一个分布，它是一个**mark 形状**（描边弧），它的分布是「同心环」。
同理 ORBS 的 `tilt` / `depth` 属于「轨道」这个分布，它的球体加光晕是一个 mark 形状。

### 所以合并后的形状

**一个点标记模式**，取代 DOTS + ARCS + ORBS + letter-shape **整个工具**：

- **分布 4 个**：`grid`（来自 DOTS）· `rings`（ARCS）· `orbit`（ORBS）· `glyph`（letter）
  ——**任意两个之间可以 morph**
- **mark 形状 6 个**：`dot` · `arc` · `sphere` · `brick` · `ascii` · `ring`
  ——**独立的一个轴**，所以点阵可以用 ASCII 画、字形可以用描边弧画

**四个模式保留自己的图元**：FIELD · FLOW · AURA · SLICE。
它们继续用 cream 现有的 `build / place / draw / toSVG` 契约——那个契约对它们是对的。

净结果：cream 七个模式 + letter 一个工具 → **五个模式**（1 个点标记 + 4 个图元专用），
而那个点标记模式比它取代的三个加起来表达力强得多。

### 合并之后白拿的三样

1. **任意两个点分布之间可以 morph。**
   letter 的炸开重组本质就是重跑 sampler 再 tween `from → to`。sampler 一旦可插拔，
   `grid → glyph`、`rings → glyph`、`orbit → grid` 全都成立。**两个 repo 现在都做不到。**
   （只在点标记模式内部成立。FIELD / FLOW / AURA / SLICE 是别的图元，不参与。）
2. **分布 × mark 形状变成两个独立的轴。**
   cream 的点阵可以用 ASCII 画，letter 的字形可以用圆环画。现在两边都表达不了。
3. **字形拿到 SVG 导出。**
   cream 的 `toSVG()` 就是遍历点列表发几何——它不关心点从哪来。

### 以及互相补的两样

- **Marks 拿到代码导出。** letter 用 `Function.prototype.toString()` 把活引擎烤进
  Vanilla / React / Vue，所以线上跑的和工具里看到的是同一套逻辑。cream 没有这个。
- **字形拿到麦克风。** cream 的三频段音频（bass → size / mid → speed / hi → jitter）
  驱动的是**逐帧参数**而不是布局，所以它挂在 `place(t)` 上，对所有分布生效，字形也在内。

---

## Mesh 的位置：它不是那两个的同辈，它是它俩踩的地

Mesh 产**连续场**，Marks 产**离散元素**。这不是两个竞争的模式，是一张图的两半。

cream 的背景现在是一个纯色，letter 是纯色加六个预设。换成 mesh 场，
「Mesh 在这个 app 里干嘛」立刻有答案：**它是底，marks 落在上面。**

**所以 Studio 不该有顶层三选一。** 它只该有一个图层栈，收两种图层。

### 这逼出一个必须做的决定

Mesh 的颜色是每个 `MeshNode` 一个 hex；cream 和 letter 是**加权色板**。
一个文档里你要**一套**色板被所有图层读，所以 **mesh 节点必须改成引用色板槽位而不是字面量**。

这动 meshy 的 `types/gradient.ts`、129 个预设、工程文件格式和节点检查器。
不是能悄悄做掉的重构。但它正是让混合构图看起来像「构图」而不是「拼贴」的那件事。

---

## 统一的 IA：六格，按管线顺序

```
① Source   文档级   底下是什么
② Layers   文档级   有几个 ③，怎么叠
③ Subject  图层级   引擎独占 ← 唯一真分歧
④ Motion   图层级   一个 transport，per-layer speed
⑤ Finish   文档级   全局后期
⑥ Frame    文档级   画板
```

两个产品都填得满这六格。Studio 叠、Glass 链；②的标题从 Layers 变成 Panes，
blend 的标签从 Blend 变成 Re-emit as，左栏从 Presets 变成 Templates。其余是同一份实现。

⑤ **在代码里已经是共享的**：glass-studio 的 `EffectsSettings` 把 meshy 那 22 个字段
（`grain` 到 `backdropGlows`）逐字段抄了过去，它的 types 文件自己写着
*"Inherited wholesale from the mesh engine this studio grew out of"*。

### 三条规则做实际的统一工作

**规则 1 — 每个 section 只有一个作用域。**
文档级和图层级永不交错。meshy 现在最伤的地方就是这个：九个平铺 section 里
Layers / Point / Color / Animation 是 per-layer，Effects / Pattern / Backdrop glow / Background
是全局，UI 上没有任何东西说明。

**规则 2 — 每个 Subject 都以 Kind picker 开头。**
Glass 18 shapes、Marks 5 modes（点标记模式内部再有 4 个分布）、Field 2 topologies——
这是同一个控件的三个名字。
做成字面上同一个组件。这条同时保证 **cream 的七个模式不升到顶层**：它们是 Marks 的 Kind，
和 Glass 的 18 个 shape 同级。

**规则 3 — 每个 section 内部三层。**
Primary（≤4 个）→ *More* → *Settings*。cream 已经这么跑了（`pri:true` 标记），
泛化一下就是。

### 一个要点名的不对称

**Field 层在画布上有东西可拖，Marks 层没有。**
meshy 有节点和贝塞尔手柄；glass 明确站在反面——
*"There is deliberately nothing to drag on the canvas. Glass is a material you dial in,
not a mesh you sculpt."* cream 和 letter 站 glass 那边。

这不是冲突，是**选择模型**：选中 Field 层露出网格，选中 Marks 层什么都不露。
跟选中矢量出手柄、选中图片不出，是同一件事。

---

## 分期

| | 做什么 | 为什么是这个顺序 |
|---|---|---|
| **Phase 0** | 抽 `packages/ui` | `Slider` `Section` `Segmented` `Switch` `GlassPanel` `ColorField` `Kbd` 在 meshy 和 glass 里**逐字节相同**，`Button` 差 5 行。零风险、可回退。Glass 虽然独立，也用这套壳。 |
| **Phase 1** | **cream + letter → 一个 Marks 引擎**：DOTS / ARCS / ORBS 和 letter 整个工具合成**一个点标记模式**（4 分布 × 6 形状 + morph）；FIELD / FLOW / AURA / SLICE 原样保留 | **先做这个，不碰 Mesh。** 两个都是零依赖 canvas 2D，这一阶段还能以单文件交付，双击就跑的属性保得住。而且它单独就交付了上面「白拿的三样 + 互补的两样」。 |
| **Phase 2** | Studio：Field + Marks 进同一个文档 | 两个决定扛这一阶段：`renderAt(t) → HTMLCanvasElement` 边界（让 WebGL 层合成进 2D 栈，同时让图片/视频/GIF 导出只写一次），和 mesh 节点改色板槽位。**单文件属性在这里花掉。** |
| **Phase 3** | Studio ↔ Glass 交接 | `studio-doc` source kind + Open in Glass。顺带统一导出编码器，见下。 |

### Phase 3 顺带要收的：三份 GIF 编码器

四个 repo 里有**三份**独立手写的 GIF89a（cream、letter、glass），**glass 那份最好**——
中位切分调色板（另两份是 RGB555 桶采样）、跨所有帧一份全局调色板（理由写在文件头：
逐帧调色板会闪）、Floyd–Steinberg 可关默认开。合并时留 `glass-studio/lib/gif.ts`。

glass 还有 `lib/mp4.ts`：`VideoEncoder`（WebCodecs）加手写的经典布局 MP4 muxer
（一个 `mdat` 加完整 sample table，Premiere / Final Cut 能直接导入）。

**meshy 是唯一一个既没有 GIF 也没有 MP4 的**——它只有 `MediaRecorder` 的 WebM，
而 `MESHY_NOTES.md` 自己标了那是实时录制、高分辨率根本跟不上。
两个编码器都进 `packages/core`，四条产品线一起拿到。

目标结构：

```
studio/
├── apps/studio            Field + Marks，一个文档，混合栈，WebGL + 2D 双宿主
├── apps/glass             独立产品，自己的左栏，链式面
├── packages/ui            Slider Section Segmented Switch GlassPanel ColorField Kbd Dialog
├── packages/core          文档信封 · transport · history · seeded rng · renderAt() · GIF + MP4
├── packages/finish        那 22 个字段的后期链，WebGL pass + 2D pass
├── packages/engine-field   ← meshy-studio                      · WebGL
├── packages/engine-marks   ← cream-studio + letter-shape       · 2D · 5 modes
└── packages/engine-glass   ← glass-studio                      · WebGL · chain
```

---

## 代价（诚实清单）

1. **Studio 留不住单文件。**
   cream 和 letter 现在双击 `index.html` 就跑、零外部请求、离线可用。Mesh 要 three.js。
   Phase 1 保得住，Phase 2 花掉。如果这个属性重要，答案是**每个 app 一个静态导出目标**，
   不是继续维持四个 repo。

2. **cream 的 visionOS 表面留不住原样。**
   它的对比度是对着一张具体背景图**解出来**的——`--scrim` 从那张图最亮处 153 反解，
   `PRODUCT.md` 第 10 条明确警告换图必须重算。进统一 token 之后整套推导作废。
   **留成 theme，不要留成第二套 IA**，然后重解一次。

3. **Mesh 的颜色模型必须改。** 见上面「必须做的决定」。

4. **Presets / Looks / Templates 变成两条栏而不是一个词。**
   Studio 的栏合并 meshy 的 129 + cream 的 35 + letter 的 5，Glass 保留自己的 40。
   glass 的三个动词（replace / stack `+` / shuffle）应该是两边的模型。

---

## 不要「修正」回去的东西

- **不要改 SKILL.md。** cream 和 letter 的规范原件仍是参数默认值与架构的权威，
  `redskill upgrade` 会整份覆盖。对它的偏离逐条记在各自的 `PRODUCT.md` 里。
- **确定性是硬规则。** 同 seed 同构图。`Math.random()` 只允许出现在 reseed、随机配色、
  音频逐帧抖动这几处。合并后 seed 升到文档级；Field 层不用 seed（它靠显式参数确定），
  这不是缺陷。
- **不要引入科技感 / 霓虹 / 高饱和色板。** letter 的 SKILL.md 记录这个方向被明确否决
  并回退过两次，禁令仍然有效。
- **"There is deliberately nothing to drag on the canvas" 是立场，不是遗漏。**
  Marks 层不要加画布直接操作。
- **一个文档一个 transport。** meshy 和 glass 用同一句话辩护过：per-layer 的
  play/pause 只会让人不小心把图层弄得不同步；per-layer **speed** 才是让图层错开的东西。
