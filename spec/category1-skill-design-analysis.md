# Category 1 Skill 设计思路分析：Document & Asset Creation

## 分析对象

本文基于仓库中四个典型的 Category 1 skills，分析其设计思路与白皮书 key techniques 的对应关系：

| Skill | 类型 |
|-------|------|
| `frontend-design` | 纯提示词，无脚本 |
| `brand-guidelines` | 含品牌规范数据 |
| `canvas-design` | 多阶段工作流 |
| `doc-coauthoring` | 结构化协作流程 |

---

## 核心洞察：这类 Skill 的本质

Category 1 的 skill 做的是一件事：**把"什么是好的输出"这个判断从 Claude 身上拿走，直接嵌入指令里**。

传统方式是"告诉 Claude 这类任务的目标"，Category 1 的方式是"把所有审美/结构/质量标准直接写进 Claude 的上下文"，让 Claude 只需专注于内容本身，不再需要猜测用户的品味或标准。

---

## 四项 Key Techniques 的设计分析

---

### Technique 1: Embedded Style Guides and Brand Standards（嵌入式风格指南）

**设计思路**：不是引用外部文档，而是把标准直接写进 SKILL.md

#### `brand-guidelines` —— 最字面的实现

```markdown
### Colors
- Dark: `#141413` - Primary text and dark backgrounds
- Light: `#faf9f5` - Light backgrounds and text on dark
- Orange: `#d97757` - Primary accent

### Typography
- Headings: Poppins (with Arial fallback)
- Body Text: Lora (with Georgia fallback)
```

这里连 hex 色值都写死了。Claude 调用这个 skill 时，不需要去查任何外部资料，颜色和字体的品牌标准就在上下文里。

#### `frontend-design` —— 用"反向禁止"实现风格标准

```markdown
NEVER use generic AI-generated aesthetics like overused font families
(Inter, Roboto, Arial, system fonts), cliched color schemes
(particularly purple gradients on white backgrounds)...
```

这是一个聪明的设计：不列出"要用什么"（选择太多），而是列出"不能用什么"（范围可枚举）。通过排除法，驱动 Claude 去寻找差异化方案。

**设计原理**：
- 风格指南不应该是外链或参考文档，而是直接嵌入上下文的具体规则
- 可以用正向规定（用 Poppins）或反向禁止（不用 Inter）两种形式
- 越具体越好——hex 值比"使用品牌色"更有效

---

### Technique 2: Template Structures for Consistent Output（模板结构）

**设计思路**：模板不一定是文件模板，可以是认知框架或工作流阶段

#### `canvas-design` —— 强制两阶段输出结构

```markdown
Complete this in two steps:
1. Design Philosophy Creation (.md file)
2. Express by creating it on a canvas (.pdf or .png)
```

每个阶段有详细子结构（命名运动、四到六段哲学表述、视觉表达），确保 Claude 每次都先想清楚"要做什么风格"，再动手，而不是直接开始生成。

#### `doc-coauthoring` —— 三阶段协作流程模板

```markdown
Stage 1: Context Gathering
Stage 2: Refinement & Structure
Stage 3: Reader Testing
```

每个阶段内部还有子步骤（Clarifying Questions → Brainstorming → Curation → Drafting → Refinement）。这是一个**过程模板**，约束的是 Claude 的工作顺序，而非输出格式。

#### `frontend-design` —— 最轻量的认知模板

```markdown
Before coding, understand the context:
- Purpose: What problem does this interface solve?
- Tone: Pick an extreme...
- Constraints: Technical requirements...
- Differentiation: What makes this UNFORGETTABLE?
```

纯粹的"思维框架"，没有格式约束，只有思考顺序约束。

**设计原理**：
- 模板的粒度要匹配任务复杂度：简单任务用认知框架，复杂任务用多阶段工作流
- 模板的作用是约束"生产顺序"，防止 Claude 跳步（如直接生成而不先理解需求）
- 输出格式要求（`.md`、`.pdf`、`.png`）也是模板的一部分

---

### Technique 3: Quality Checklists Before Finalizing（最终确认前的质量检查）

**设计思路**：质量检查不以 `[ ]` 列表的形式出现，而是作为**流程中的强制停顿点**嵌入

#### `canvas-design` —— 最明确的二次精修阶段

```markdown
## FINAL STEP

IMPORTANT: The user ALREADY said "It isn't perfect enough. It must be
pristine, a masterpiece of craftsmanship, as if it were about to be
displayed in a museum."

Take a second pass. Go back to the code and refine/polish further...
```

这里用了一个聪明的技巧：把"用户会不满意"这个预期直接写进指令。Claude 第一次生成后，被强制触发一次质量反思，不管用户实际上有没有抱怨。

另一个质量门控：

```markdown
avoid adding more graphics; instead refine what has been created.
If the instinct is to call a new function or draw a new shape, STOP and
ask: "How can I make what's already here more of a piece of art?"
```

这是对 Claude 常见行为失误的预防——当 Claude 想通过"加东西"来改善时，指令强制它转向"精炼现有内容"。

#### `doc-coauthoring` —— 把质量检查做成独立 Stage

Reader Testing 阶段本质上就是质量检查：用"无上下文的新 Claude"来测试文档，等同于模拟真实读者。

```markdown
When Reader Claude consistently answers questions correctly and doesn't
surface new gaps or ambiguities, the doc is ready.
```

这是一个明确的"准出"标准（exit condition）——不达标不能结束。

#### `frontend-design` —— 用语气强度实现隐式检查

```markdown
CRITICAL: Choose a clear conceptual direction...
IMPORTANT: Match implementation complexity to the aesthetic vision...
Differentiation: What makes this UNFORGETTABLE?
```

用 CRITICAL/IMPORTANT 等标记作为隐式检查点，迫使 Claude 在关键节点上自我审查。

**设计原理**：
- 质量检查的有效形式：强制二次 pass（canvas-design）、独立验证阶段（doc-coauthoring）、强制语气标记（frontend-design）
- 最有效的质量检查是在**流程设计层面**制造停顿，而不是在结尾加一句"请检查质量"
- "预设用户会不满意"比"请确保质量"更有执行力

---

### Technique 4: No External Tools Required（无需外部工具）

**设计思路**：以 Claude 的内置能力为核心，外部工具最多作为可选增强

#### `frontend-design` —— 极简，零依赖

```
frontend-design/
├── SKILL.md
└── LICENSE.txt
```

HTML/CSS/JS/React 都是 Claude 的内置知识。唯一提到的外部库（Motion library）明确标注为"when available"。

#### `doc-coauthoring` —— 外部工具作可选增强

```markdown
If integrations are available (e.g., Slack, Teams, Google Drive, SharePoint,
or other MCP servers), mention that these can be used to pull in context directly.

If no integrations are detected... Suggest they can enable connectors...
```

设计模式：**先定义无外部工具时的完整工作流，再把外部工具作为可选加速器叠加上去**。核心流程不依赖任何外部服务。

#### `brand-guidelines` —— 对"外部工具"的重新理解

```markdown
No font installation required - works with existing system fonts
For best results, pre-install Poppins and Lora fonts in your environment
```

"no external tools"的含义是**不要求用户安装额外服务或获取 API key**，而不是"绝对不能有任何依赖"。系统字体这类环境原生能力是允许的。

**设计原理**：
- Skill 应该在"无任何特殊环境"下就能工作
- 外部工具（MCP、连接器）可以作为可选增强，但不能作为必须依赖
- 这个设计使 skill 具有最大的可移植性（claude.ai、API、IDE 插件都能用）

---

## 整体设计哲学：为什么四个技术点"看不见"

四个 key techniques 都不以独立的结构化文件出现，而是**融合在自然语言指令中**。这不是偶然，是刻意的设计选择：

| 传统软件的实现方式 | Skill 的实现方式 |
|----------------|----------------|
| 独立的 style guide 文件 | 嵌入 SKILL.md 正文的行为规则 |
| 模板引擎 + `.template` 文件 | 步骤序列 + 认知框架 |
| 独立的 linter / validator | CRITICAL 标记 + 强制二次 pass |
| SDK / 外部 API | Claude 的内置知识 |

**核心原因**：Skill 的"执行引擎"是 Claude 自身的语言理解能力。指令即工具，文档即程序。把所有约束写成自然语言，反而比写成独立系统更有执行力——因为 Claude 能理解"意图"，而不只是匹配规则。

这就是 Category 1 的设计范式：**把领域专家的判断力编码进提示词，让 Claude 替代领域专家的角色**。
