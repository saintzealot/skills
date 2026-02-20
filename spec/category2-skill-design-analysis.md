# Category 2 Skill 设计思路分析：Creative & Design

## 分析对象

README 分组中的 Creative & Design 示例：

| Skill | 文件 | 核心输出形式 |
|-------|------|-------------|
| `algorithmic-art` | `skills/algorithmic-art/SKILL.md` | HTML（内嵌 p5.js canvas 交互式艺术） |
| `canvas-design` | `skills/canvas-design/SKILL.md` | PNG / PDF 静态视觉设计 |
| `brand-guidelines` | `skills/brand-guidelines/SKILL.md` | 应用到任意 artifact 的样式层 |
| `theme-factory` | `skills/theme-factory/SKILL.md` | 主题规格 → 样式化后的 artifact |
| `slack-gif-creator` | `skills/slack-gif-creator/SKILL.md` | 优化为 Slack 的动画 GIF |

---

## 核心洞察：Creative & Design 的两个子模式

| 子模式 | 代表 | 特征 |
|--------|------|------|
| **哲学驱动生成** | `algorithmic-art` / `canvas-design` | 先写 4-6 段哲学宣言再生成代码/图像，思想先行 |
| **系统化应用** | `brand-guidelines` / `theme-factory` / `slack-gif-creator` | 给定规格集合，系统化地应用到输出 |

与 Category 1（Document & Asset Creation）的最大区别：Category 1 的风格指南是**领域技术规范**（DXA 单位、XML 元素顺序），Category 2 的风格指南是**美学主张**（什么是有灵魂的设计，什么是 AI 味）。

---

## 四项 Key Techniques 的设计分析

---

### Technique 1: Embedded Style Guides and Brand Standards（嵌入式风格指南）

**设计思路**：Category 2 的风格指南不是"用什么规格"，而是"怎么想"——把美学判断力和哲学立场内化进 SKILL.md。

#### `algorithmic-art` —— 把创作哲学本身变成工作流的第一步

```markdown
An algorithmic art piece should embody a distinct philosophy—
a unique perspective or emotional truth that the algorithm expresses.
The philosophy should be specific enough to guide implementation choices
but abstract enough to allow genuine surprise in the output.
```

这不是告诉 Claude "用什么颜色"，而是要求 Claude 在写代码之前先确立一个**可以指导实现选择**的哲学立场。

具体实现方面，SKILL.md 规定了 Anthropic 品牌色彩（这是美学约束）：
```
Colors: #141413, #faf9f5, #b0aea5, #e8e6dc
Accent: #d97757 (orange)
Fonts: Poppins (headings), Lora (body)
```

同时有反向禁止：
```markdown
Create original algorithmic art rather than copying existing artists' work
to avoid copyright violations.
```

#### `canvas-design` —— 用设计运动（Design Movement）驱动风格

```markdown
NEVER create decorative art or illustrations — create design.
Visual-first approach: 90% visual, 10% essential text only.
Ideas are communicated through form, color, and composition — not paragraphs.
```

设计运动示例体现了具体的美学立场：
```
"Brutalist Joy" — brutal geometry that somehow feels warm and playful
"Chromatic Silence" — color as the primary language, letting color breathe
"Metabolist Dreams" — modular and expandable forms
"Concrete Poetry" — text as visual element, not information carrier
```

这些命名不是菜单——它们示范了一种**命名自己创造的设计语言**的方式。

#### `brand-guidelines` —— 精确到 RGB 的品牌系统

```markdown
Dark: #141413 (RGB: 20,20,19)
Light: #faf9f5 (RGB: 250,249,245)
Accent Orange: #d97757 (RGB: 217,119,87)
Accent Blue: #6a9bcc (RGB: 106,155,204)
Accent Green: #788c5d (RGB: 120,140,93)
Typography: Poppins 24pt+ → headings, Lora → body
Fallback: Arial → headings, Georgia → body
```

Color 精确到 RGB，有备用字体策略，不依赖用户额外安装字体。

#### `theme-factory` —— 10 个命名主题作为具体美学立场

```markdown
Ocean Depths — Professional and calming maritime theme
Sunset Boulevard — Warm and vibrant sunset colors
Forest Canopy — Natural and grounded earth tones
Modern Minimalist — Clean and contemporary grayscale
...
Midnight Galaxy — Dramatic and cosmic deep tones
```

每个主题是一个**完整的美学立场**（色彩 + 字体 + 视觉意象），不是参数集合。主题名称本身就携带美学方向。

#### `slack-gif-creator` —— 技术规格作为创意约束

```markdown
Emoji GIFs: 128x128px, <3 seconds, 48-128 colors
Message GIFs: 480x480px, 10-30 FPS
Avoid low-contrast elements — icons AND text need strong contrast
```

Slack 的技术限制（文件大小、色彩数量）被编码为**创意约束**，驱动工具选择（PIL 而不是高分辨率渲染）。

**设计原理**：
- Category 2 的风格指南是**美学判断的内化**，而非技术规范的传递
- 具名的设计运动（"Brutalist Joy"）比抽象形容词（"bold"）更有执行力
- 正向的品牌色系（hex + RGB）和反向的禁止项（"never copy existing artists"）结合使用
- 技术限制可以转化为创意框架（Slack 规格 → 动画设计约束）

---

### Technique 2: Template Structures for Consistent Output（模板结构）

**设计思路**：Category 2 的模板不是格式要求，而是**创作工作流**的锚定点，保证 Claude 不会随机发挥。

#### `algorithmic-art` —— HTML 模板 + JS 代码组织模板（最具体）

提供两个独立模板文件：
```
templates/viewer.html     ← CRITICAL: 所有 artifact 的起点，结构不可修改
templates/generator_template.js  ← 代码组织模式参考
```

`viewer.html` 规定了固定的三段式 UI 结构：
```
Seed controls (fixed) → Parameters (variable) → Actions (fixed)
```

Claude 只需要实现中间的 Parameters 部分（参数随哲学而变化），首尾固定。这是一种**变体区间约束**：固定不变的部分保证品牌一致性，可变部分释放创意空间。

#### `canvas-design` —— 哲学宣言模板（认知框架）

没有 HTML 模板，但有强制的认知工作流：
```markdown
Philosophy (4-6 paragraphs) MUST come before visual execution.
Each design must have a named movement (e.g., "Chromatic Silence").
The movement name should be specific enough to guide all visual decisions.
```

工作流模板：
```
1. 命名设计运动
2. 写 4-6 段哲学（具体到可以指导实现）
3. 执行视觉
4. 迭代精炼（不是添加元素，而是精炼现有构图）
```

"迭代精炼"而非"添加元素"是重要的约束——它防止 Claude 通过堆砌来填满画面。

#### `theme-factory` —— 选择路由模板（最轻量）

```markdown
## Workflow
1. Show user the theme-showcase.pdf (visual reference)
2. Ask for selection (number or name)
3. Get confirmation
4. Read corresponding theme file (ocean-depths.md etc.)
5. Apply colors and fonts throughout artifact
```

SKILL.md 是交互路由，具体规格在 `themes/` 子文件夹里按需加载。这与 `pptx` 的路由模板设计异曲同工，但多了一个"展示视觉参考 → 用户选择"的交互步骤。

#### `brand-guidelines` —— 后处理模板

```markdown
This skill applies Anthropic's brand to any artifact that has been created.
Read the target artifact, identify color and font application points,
apply brand colors systematically.
```

这不是输出模板，而是**应用顺序**的模板：先理解现有 artifact，再系统地施加品牌约束。

#### `slack-gif-creator` —— 编程 API 模板

`GIFBuilder` 提供固定的编程接口：
```python
builder = GIFBuilder(width=128, height=128, fps=15)
builder.add_frame(frame_data)
builder.save("output.gif", num_colors=64, optimize_for_emoji=True)
```

这是代码级别的模板：Claude 不需要处理 GIF 格式细节，只需要实现 `add_frame()` 的逻辑。

**设计原理**：
- 模板可以是 HTML 文件（algorithmic-art）、认知工作流（canvas-design）、交互路由（theme-factory）或 API 约定（slack-gif-creator）
- "固定框架 + 可变内容区"（algorithmic-art 的 viewer.html）是一种高效的模板设计：约束足够多以保证一致性，自由度足够大以保证创意
- 迭代约束（"精炼而非添加"）是防止创意劳动退化为堆砌的重要手段

---

### Technique 3: Quality Checklists Before Finalizing（最终确认前的质量检查）

**设计思路**：Creative & Design 的质量检查无法完全机械化（审美判断是主观的），因此多依赖**语气强化**和**可量化的技术指标**。

#### `algorithmic-art` —— 工艺要求作为隐式检查清单

```markdown
Master-level craftsmanship: Every parameter choice must be intentional.
The philosophical concept should be subtle, like a jazz musician who
references a standard without quoting it directly.
Gallery mode: Seed navigation must work (prev/next/random/jump/display).
```

"Master-level" 和 "jazz musician" 是**美学质量锚**，迫使 Claude 在完成前自问"这达到大师级别了吗"。

技术检查点是隐含的：canvas 尺寸（1200x1200）、种子系统（p5.js randomSeed/noiseSeed）、Gallery 模式功能。

#### `canvas-design` —— 迭代精炼指令作为强制停顿

```markdown
CRITICAL: Choose a clear conceptual direction and execute with precision.
Iterate: Refine the existing composition rather than adding new elements.
Museum/magazine quality target: meticulously crafted, pristine execution.
```

没有独立的 QA 段落，但"迭代精炼"指令是一个**强制返工机制**——第一版不算完成，必须至少精炼一次。

#### `slack-gif-creator` —— 可量化的技术验证

```python
validate_gif(output_path)    # 检查文件格式和帧结构
is_slack_ready(output_path)  # 检查 Slack 规格（尺寸、文件大小）
```

这是 Category 2 里最接近 Category 1（客观工具验证）的 QA 方式——GIF 的 Slack 合规性是可以机械检测的。

同时有技术优化 checklist（隐式）：
```markdown
Fewer frames → smaller file
Fewer colors (48-128) → smaller file
Remove duplicate frames
Emoji mode (128x128, <3s)
```

#### `brand-guidelines` / `theme-factory` —— 最隐式（依赖视觉判断）

没有独立 QA 机制。质量验证依赖用户在看到结果后的反馈，或 Claude 自我检查品牌色彩是否正确应用。

**设计原理**：
- 美学类任务的质量检查依赖**语气强化**（CRITICAL/Master-level）而非工具验证，因为审美无法二值化
- 可量化的技术指标（GIF 尺寸、帧数、色彩数）可以也应该用工具验证
- "迭代精炼"指令是一个重要的设计模式：强制至少一次改进循环，防止 Claude 将第一版作为最终输出

---

### Technique 4: No External Tools Required（无需外部工具）

**设计思路**：Category 2 展现了工具依赖的四种形态，从完全无依赖到打包依赖，形成一个完整的谱系。

#### `brand-guidelines` / `theme-factory` —— 零运行时依赖（最纯粹）

```
brand-guidelines/
├── SKILL.md
└── LICENSE.txt

theme-factory/
├── SKILL.md
├── theme-showcase.pdf
└── themes/*.md
```

Claude 根据 SKILL.md 中的色彩/字体规格直接修改 artifact，无需运行任何代码。theme-factory 的 `theme-showcase.pdf` 是参考文档，不是可执行资源。

#### `algorithmic-art` —— CDN 依赖（运行时加载，非打包）

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
```

p5.js 从 CDN 加载。这意味着：
- Skill 本身没有安装步骤
- 但 artifact 运行时需要网络访问
- 适合于浏览器环境中运行的输出（HTML artifact）

#### `canvas-design` —— 字体资产打包（最大的静态资产）

```
canvas-design/
├── SKILL.md
├── canvas-fonts/
│   ├── ArsenalSC-Regular.ttf
│   ├── BigShoulders-Bold.ttf
│   └── ... (29+ 字体文件)
```

字体随 skill 打包。这确保了：
- 无需网络连接
- 设计输出字体可控（29+ 专业字体 vs 系统默认字体）
- 代价是 skill 体积较大（字体文件集合）

Python 库（reportlab、Pillow）是系统级依赖，SKILL.md 假设已安装。

#### `slack-gif-creator` —— 核心模块打包（最完整的工具封装）

```
slack-gif-creator/
├── core/
│   ├── gif_builder.py   ← GIF 生成主 API
│   ├── validators.py    ← validate_gif, is_slack_ready
│   ├── easing.py        ← 动画缓动函数
│   └── frame_composer.py ← 帧工具函数
└── requirements.txt
```

核心功能完全封装在 skill 自带的 Python 模块里。Claude 只需调用 `GIFBuilder` API，不需要理解底层 GIF 格式。PIL/imageio/numpy 是系统依赖（`requirements.txt`），但无需 API key 或网络服务。

**设计原理**：

| 形态 | 代表 | 适用场景 |
|------|------|---------|
| 零依赖（纯参考） | brand-guidelines, theme-factory | 输出是文本/样式，Claude 直接应用规格 |
| CDN 依赖 | algorithmic-art | 输出是在浏览器运行的 HTML，CDN 是合理假设 |
| 静态资产打包 | canvas-design | 输出质量强依赖特定资产（字体），需随 skill 携带 |
| 代码模块打包 | slack-gif-creator | 操作复杂且重复，需封装为可调用 API |

Category 2 vs Category 1 的差异：Category 1（docx/pptx/xlsx）的工具依赖是**可执行脚本**（LibreOffice、validate.py），Category 2 更多是**资产打包**（字体）和**库封装**（GIFBuilder core），少有直接调用本地可执行程序的需求。

---

## 整体设计哲学：两个核心决策

### 决策 1：把"美学判断力"而非"技术规范"内化进 SKILL.md

Category 1 内化的是领域技术规范（Excel 色彩编码惯例、DXA 单位、XML 元素顺序）。Category 2 内化的是**美学判断力**：

| Skill | 内化了什么美学判断 |
|-------|-----------------|
| `algorithmic-art` | 什么是有哲学深度的生成艺术（jazz 类比：致敬而非引用） |
| `canvas-design` | 什么是设计 vs 装饰（90% 视觉，形式传达意义） |
| `brand-guidelines` | Anthropic 的视觉语言（具体到 RGB 的精确定义） |
| `theme-factory` | 10 种完整的美学立场（命名的主题比参数集更有表达力） |
| `slack-gif-creator` | 什么是高质量动画（厚线条、互补色、缓动函数） |

**结果**：Claude 不需要猜"什么样的创意输出是好的"，因为好的标准已经以美学语言的形式写进了上下文。

### 决策 2：创作工作流的"分阶段锚定"

Category 2 的多个 skills 都体现了**分阶段强制停顿**的设计：

```
algorithmic-art: 哲学确立 → 参数设计 → 代码实现 → Gallery 验证
canvas-design: 运动命名 → 哲学写作 → 视觉执行 → 精炼迭代
theme-factory: 展示参考 → 用户选择 → 确认 → 读取规格 → 应用
```

每个阶段的完成是下一阶段的前提，防止 Claude 跳过思考直接输出。这与 Category 1（xlsx 的 MANDATORY 步骤）的思路一致，但在创意类任务中，被保护的不是"公式重算"而是"哲学深度"。

**Category 2 的设计范式** = 把"美学判断力"内化进 SKILL.md + 用"分阶段工作流"保护创作过程的深度。
