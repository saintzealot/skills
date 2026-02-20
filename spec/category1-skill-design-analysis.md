# Category 1 Skill 设计思路分析：Document & Asset Creation（修订版）

## 分析对象

白皮书原文明确指定的 Category 1 示例：

| Skill | 文件 | 类型 |
|-------|------|------|
| `frontend-design` | `skills/frontend-design/SKILL.md` | **主示例**，纯 Claude 能力，无外部依赖 |
| `docx` | `skills/docx/SKILL.md` | 工具增强型，含 `scripts/` 目录 |
| `pptx` | `skills/pptx/SKILL.md` | 工具增强型，含 `editing.md` / `pptxgenjs.md` |
| `xlsx` | `skills/xlsx/SKILL.md` | 工具增强型，含 `scripts/` 目录 |

---

## 核心洞察：两种子模式

Category 1 内部存在两个子模式：

| 子模式 | 代表 | 特征 |
|--------|------|------|
| **纯 Claude 生成** | `frontend-design` | SKILL.md + LICENSE.txt，Claude 直接输出 HTML/CSS/JS |
| **工具增强生成** | `docx` / `pptx` / `xlsx` | SKILL.md + `scripts/`，Claude 编排本地工具链 |

白皮书说的 "No external tools required" 最纯粹地体现在 `frontend-design`；docx/pptx/xlsx 的工具是**随 skill 打包**的（scripts/ 目录），不要求用户额外购买或配置外部服务。

---

## 四项 Key Techniques 的设计分析

---

### Technique 1: Embedded Style Guides and Brand Standards（嵌入式风格指南）

**设计思路**：不是说"请遵循最佳实践"，而是把标准直接写进 SKILL.md，且标准是**领域专属**的。

#### `frontend-design` —— 用"反向禁止"实现创意风格指南

```markdown
NEVER use generic AI-generated aesthetics like overused font families
(Inter, Roboto, Arial, system fonts), cliched color schemes
(particularly purple gradients on white backgrounds), predictable
layouts and component patterns...
```

正向规定（"用什么"）在创意类任务中难以穷举，所以改用**排除法**——列出不能用的，驱动 Claude 主动寻找差异化方案。

#### `pptx` —— 用具体数据表格实现设计规范

```markdown
| Theme           | Primary         | Secondary       | Accent        |
| Midnight Exec   | `1E2761` (navy) | `CADCFC` (ice)  | `FFFFFF`      |
| Forest & Moss   | `2C5F2D`        | `97BC62`        | `F5F5F5`      |

| Header Font | Body Font |
| Georgia     | Calibri   |
| Arial Black | Arial     |

| Element     | Size         |
| Slide title | 36-44pt bold |
| Body text   | 14-16pt      |
```

直接给出 hex 色值、字体配对、字号标准。Claude 生成幻灯片时不需要"猜"应该用什么颜色。

同时也有反向禁止：
```
NEVER use accent lines under titles — these are a hallmark of AI-generated slides
```

#### `xlsx` —— 嵌入**行业标准**的编码惯例

```markdown
- Blue text (RGB: 0,0,255): Hardcoded inputs
- Black text (RGB: 0,0,0): ALL formulas and calculations
- Green text (RGB: 0,128,0): Links from other worksheets
- Red text (RGB: 255,0,0): External links to other files
- Yellow background: Key assumptions needing attention
```

这是金融建模行业的行业标准色彩语义，不是 Anthropic 自定义的。Skill 把行业专业知识编码进来，让 Claude 像金融分析师一样工作。

#### `docx` —— 嵌入排版规范（含字符实体）

```markdown
Use Arial as the default font (universally supported)
Use smart quotes: &#x2019; (apostrophe), &#x201C; (left double)
Use "Claude" as the author for tracked changes
```

连引号的 XML 实体都规定好了，排版规范细化到字符级别。

**设计原理**：
- 风格指南是**领域专属**的：创意类用美学偏好，金融类用行业色彩惯例，文档类用排版规范
- 正向规定（表格形式的调色板）和反向禁止（NEVER 列表）通常结合使用
- 越具体越好——hex 值、字符实体、pt 大小比模糊原则更有执行力

---

### Technique 2: Template Structures for Consistent Output（模板结构）

**设计思路**：模板在这里有三种形式：认知框架、工作流模板、代码模板。

#### `frontend-design` —— 认知框架（最轻量）

```markdown
Before coding, understand the context and commit to a BOLD aesthetic direction:
- Purpose: What problem does this interface solve? Who uses it?
- Tone: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic...
- Constraints: Technical requirements (framework, performance, accessibility)
- Differentiation: What makes this UNFORGETTABLE?
```

纯粹的**思考顺序约束**，没有输出格式要求，只是强制 Claude 在动手前想清楚四件事。

#### `docx` —— 代码模板（最具体）

SKILL.md 中直接给出可复用的 JavaScript 代码片段作为模板：

```javascript
const doc = new Document({
  styles: { paragraphStyles: [
    { id: "Heading1", name: "Heading 1",
      run: { size: 32, bold: true, font: "Arial" },
      paragraph: { outlineLevel: 0 } },
  ]},
  numbering: { config: [
    { reference: "bullets",
      levels: [{ format: LevelFormat.BULLET, text: "•" }] }
  ]},
  sections: [{
    properties: { page: { size: { width: 12240, height: 15840 } } },
    children: []  // ← Claude 只需填这里
  }]
});
```

这不是说明文档，这是**直接可用的代码模板**，Claude 只需填入内容部分。

#### `pptx` —— 路由模板（最有意思的设计）

`pptx` 的 SKILL.md 本身是一个**路由器**，不包含操作细节，而是指向两个专门文件：

```markdown
| Task                | Guide          |
| Edit from template  | Read editing.md   |
| Create from scratch | Read pptxgenjs.md |
```

这是模板结构的元设计：SKILL.md 是导航层，复杂细节拆到子文件里，避免单文件过长。

#### `xlsx` —— 工作流模板（带强制检查点）

```markdown
## Common Workflow
1. Choose tool: pandas for data, openpyxl for formulas/formatting
2. Create/Load
3. Modify
4. Save
5. Recalculate formulas (MANDATORY IF USING FORMULAS): python scripts/recalc.py output.xlsx
6. Verify and fix any errors
```

第 5 步标注了"MANDATORY"——这是一个强制停顿点，不能跳过。

**设计原理**：
- 认知框架（frontend-design）：约束思考顺序，强制先想再做
- 代码模板（docx）：直接给可复用代码，减少从头构建的认知负担
- 路由模板（pptx）：把复杂细节拆到子文件，主 SKILL.md 保持简洁可扫描
- 工作流模板（xlsx）：约束执行步骤，MANDATORY 确保不跳过关键环节

---

### Technique 3: Quality Checklists Before Finalizing（最终确认前的质量检查）

**设计思路**：质量检查从最隐式到最显式都有，形态取决于任务的可机械验证程度。

#### `frontend-design` —— 最隐式（语气强度作为隐式检查）

没有独立的 QA 段落，质量要求嵌入在关键规则里：
```markdown
CRITICAL: Choose a clear conceptual direction and execute it with precision.
IMPORTANT: Match implementation complexity to the aesthetic vision.
Meticulously refined in every detail
```
CRITICAL/IMPORTANT 标记迫使 Claude 在关键节点自我确认。

#### `docx` —— 工具验证型检查

```markdown
After creating the file, validate it.
python scripts/office/validate.py doc.docx
If validation fails, unpack, fix the XML, and repack.
```

质量检查通过**工具执行**，不依赖 Claude 自我评估，pass/fail 标准客观明确。

#### `pptx` —— 最完整的显式 QA 流程

有独立的 "QA (Required)" 段落，开宗明义：
```markdown
Assume there are problems. Your job is to find them.
Your first render is almost never correct.
```

包含三层检查：
- **Content QA**：`python -m markitdown output.pptx` 文本提取检查
- **Visual QA**：转图片后用 **subagent** 做视觉检查
  - 关键设计：专门用 subagent，因为主 Claude 已"盯着代码看"，存在确认偏误
- **Verification Loop**：明确迭代循环 + 退出条件

```markdown
1. Generate → Convert to images → Inspect
2. List issues found (if none, look again more critically)
3. Fix issues
4. Re-verify affected slides
5. Repeat until full pass reveals no new issues

Do not declare success until you've completed at least one fix-and-verify cycle.
```

#### `xlsx` —— 最字面的 `[ ]` Checklist

```markdown
## Formula Verification Checklist

### Essential Verification
- [ ] Test 2-3 sample references
- [ ] Column mapping: column 64 = BL, not BK
- [ ] Row offset: Excel rows are 1-indexed

### Common Pitfalls
- [ ] NaN handling
- [ ] Division by zero (#DIV/0!)
- [ ] Wrong references (#REF!)
- [ ] Cross-sheet references: use Sheet1!A1 format

### Formula Testing Strategy
- [ ] Start small: test 2-3 cells before applying broadly
- [ ] Test edge cases: zero, negative, very large values
```

这是真正的 `[ ]` 格式 checklist，直接对应白皮书说的 "Quality checklists before finalizing"。

**设计原理**：
- 质量检查形态：语气标记 → 工具验证 → 完整 QA 流程 → `[ ]` checklist
- 形态取决于**可机械验证程度**：xlsx 公式错误客观可检测，pptx 视觉质量需要视觉判断
- pptx 用 subagent 做 Visual QA，是对"确认偏误"的明确解法
- 退出条件（`Do not declare success until...`）防止 Claude 草草收尾

---

### Technique 4: No External Tools Required（无需外部工具）

**设计思路**：区分"外部服务"和"随 skill 打包的工具"。

#### `frontend-design` —— 最纯粹：零依赖

```
frontend-design/
├── SKILL.md
└── LICENSE.txt
```

Claude 生成 HTML/CSS/JS，这是 Claude 的内置知识。唯一外部库 Motion library 标注为 "when available"，不是强制依赖。

#### `docx` / `pptx` / `xlsx` —— 有依赖，但依赖被打包进 skill

这三个 skills 都有真实的系统依赖（LibreOffice、pandoc、npm 包等），但注意：

1. **脚本随 skill 打包**：`scripts/office/soffice.py`、`scripts/recalc.py` 是 skill 自带的
2. **环境自动配置**：
   ```
   LibreOffice Required... You can assume LibreOffice is installed.
   The script automatically configures LibreOffice on first run,
   including in sandboxed environments
   ```
3. **无外部 API/服务**：没有 API key 要求，没有第三方 AI 服务调用，没有网络服务依赖

**"No external tools" 的真正含义**：
- 不需要用户注册账号、获取 API key 或订阅服务
- 所有工具要么是 Claude 内置知识（HTML/CSS），要么随 skill 打包（scripts/）
- 对比 Category 3（Workflow Automation）skills，那些需要 Slack/Google API 等外部授权

**设计原理**：
- 完全零依赖（frontend-design）是最理想的，可移植性最强
- 工具依赖可以接受，但前提是随 skill 打包且自动配置
- 绝不能依赖需要用户单独获取 API key 或账号的服务

---

## 整体设计哲学：两个核心决策

### 决策 1：把"好的标准"内化进 SKILL.md

这四个 skills 把领域专家的判断力编码进了 SKILL.md：

| Skill | 编码了谁的判断力 |
|-------|----------------|
| `frontend-design` | 创意总监的审美（什么是 AI 味，什么是有个性的设计） |
| `pptx` | 设计师的经验（配色支配性原则，避免重复布局） |
| `xlsx` | 金融分析师的行业规范（色彩语义、公式结构规则） |
| `docx` | 文档工程师的技术规范（DXA 单位、XML 元素顺序） |

结果：Claude 不需要猜"什么是好的输出"，标准已经在上下文里了。

### 决策 2：把"是否达标"外部化到工具或独立视角

质量验证不完全依赖 Claude 自我评估：

| Skill | 外部化方式 |
|-------|-----------|
| `docx` | `validate.py` 脚本客观验证 |
| `pptx` | subagent 视觉检查（避免确认偏误） |
| `xlsx` | `recalc.py` 公式错误扫描 |
| `frontend-design` | 最弱（HTML 可在浏览器直接渲染，用户自己能看） |

**Category 1 的设计范式** = 把"好的标准"内化进 SKILL.md + 把"是否达标"外部化到工具或独立视角。
