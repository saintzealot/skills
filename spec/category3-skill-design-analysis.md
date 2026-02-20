# Category 3 Skill 设计思路分析：Enterprise & Communication

## 分析对象

README 分组中的 Enterprise & Communication 示例：

| Skill | 文件 | 核心能力 |
|-------|------|---------|
| `internal-comms` | `skills/internal-comms/SKILL.md` | 公司内部通信写作（状态报告、领导更新、通讯等） |
| `doc-coauthoring` | `skills/doc-coauthoring/SKILL.md` | 三阶段协作文档工作流（文档共创） |

---

## 核心洞察：两种子模式

| 子模式 | 代表 | 特征 |
|--------|------|------|
| **格式路由型** | `internal-comms` | 识别通信类型 → 加载对应模板 → 生成输出 |
| **过程协作型** | `doc-coauthoring` | 引导用户经历多阶段过程，Claude 是向导而非执行者 |

与 Category 1（Document & Asset Creation）的根本区别：Category 1 的输出是**文件**（.docx/.xlsx）；Category 3 的输出更多是**过程**（协作流程）和**文本**（内部通信），不依赖文件格式工具链。

与 Category 2（Creative & Design）的区别：Category 2 的核心是美学判断；Category 3 的核心是**组织语境适配**——同样的内容，针对不同受众（CEO vs 团队成员）有不同的格式和语气要求。

---

## 四项 Key Techniques 的设计分析

---

### Technique 1: Embedded Style Guides and Brand Standards（嵌入式风格指南）

**设计思路**：Category 3 的"风格指南"不是美学规范，而是**组织特定的沟通规范**——公司用什么格式、什么框架、什么语气与内部成员沟通。

#### `internal-comms` —— 用示例文件实现通信规范的具体化

SKILL.md 本身非常简洁，规范内容全部下放到 `examples/` 子文件夹：

```
internal-comms/
├── SKILL.md
└── examples/
    ├── 3p-updates.md      ← 3P 框架（Progress/Plans/Problems）
    ├── company-newsletter.md
    ├── faq-answers.md
    └── general-comms.md   ← 兜底（其他类型）
```

3P 框架是一个具体的组织沟通规范：
```markdown
## 3P Updates Format
Progress: What did we accomplish since last update?
Plans: What are we planning to do next?
Problems: What blockers or risks do we face?
```

这个框架不是 Claude 自己发明的，而是公司约定俗成的——Skill 把这个约定内化进来，让 Claude 像了解公司文化的员工一样工作。

通信规范的**语气要求**也是内化的（而非让 Claude 猜）：newsletter 和 leadership update 的语气、密度、假设读者的知识水平都不同，分别在对应文件中规定。

#### `doc-coauthoring` —— 沟通对象（受众）分析作为内化的规范

```markdown
Meta-context gathering:
- Doc type: Technical spec? Decision doc? Proposal?
- Audience: Who will read this?
- Impact: What decisions does this doc enable?
- Constraints: Length, sensitivity, timeline?
```

这不是格式规范，而是**受众分析框架**——在任何创作开始之前，先系统地理解沟通对象是谁、为什么写、有什么限制。这是企业写作专家的核心能力，被内化成了标准化的上下文收集步骤。

**设计原理**：
- Category 3 的风格指南是**组织文化的编码**，而非通用写作规范
- 具体的框架（3P）比模糊的指引（"清晰、简洁"）更有执行力
- 示例文件按通信类型分开，而非合并为一个大文档——这符合按需加载（progressive disclosure）的原则

---

### Technique 2: Template Structures for Consistent Output（模板结构）

**设计思路**：Category 3 展现了两种截然不同的模板观念——一个是**输出格式的模板**，一个是**协作过程的模板**。

#### `internal-comms` —— 类型路由模板

```markdown
## Trigger Logic
1. Identify communication type from user request
2. Load corresponding example file:
   - Status/team updates → 3p-updates.md
   - Company-wide → company-newsletter.md
   - FAQ → faq-answers.md
   - Other → general-comms.md
3. Apply format from loaded file to user content
```

这是一个**分支路由**模板：不同输入走不同路径，每条路径有对应的格式规范。SKILL.md 是路由器，具体格式存在外部文件。

此设计使新增通信类型只需添加新的 example 文件，无需修改核心 SKILL.md。

#### `doc-coauthoring` —— 三阶段协作过程模板（最复杂的模板结构）

```markdown
## Stage 1: Context Gathering
- Ask meta questions (doc type, audience, impact, constraints)
- Info dump session (let user brain-dump)
- Clarify edge cases

## Stage 2: Refinement & Structure
- Build document structure first (with placeholders)
- Section-by-section: generate 5-20 options → user curates
- Iterative drafting with targeted feedback
- Use str_replace for edits (surgical, not full rewrite)

## Stage 3: Reader Testing
- Test with fresh Claude sub-agent or manual testing
- Ask specific questions a reader would have
- Iterate until reader Claude answers correctly
```

这是**多阶段协作脚本**，而非输出格式模板。每个阶段有明确的角色（Claude 是向导）、行为规范（生成选项 → 用户决策）和退出条件。

关键设计决策：
- **先建结构，再填内容**：防止 Claude 生成连续文字流，强制文档有骨架
- **5-20 选项 → 用户筛选**：把创作权交给用户，Claude 负责生成选项池
- **Surgical edits（str_replace）**：明确规定用局部替换而非重写，避免用户辛苦建立的上下文被抹掉

**设计原理**：
- 输出模板（internal-comms）适合有固定格式要求的任务
- 过程模板（doc-coauthoring）适合输出格式不固定但**过程需要结构化**的任务
- "先骨架后内容"是一个通用的写作质量保障策略——防止从第一段开始写然后迷失方向

---

### Technique 3: Quality Checklists Before Finalizing（最终确认前的质量检查）

**设计思路**：Category 3 的质量检查最有特色——`doc-coauthoring` 用 **sub-agent 扮演读者**来测试文档，这是一种独特的质量保障机制。

#### `doc-coauthoring` —— Reader Testing：用 Sub-Agent 模拟读者视角

```markdown
## Stage 3: Reader Testing

Option A: Sub-agent testing
Launch a fresh Claude instance with:
- The document
- The original audience description
Ask it to answer key questions a reader would have

Option B: Manual testing
User reads document and answers questions themselves

Exit condition:
Reader Claude consistently answers questions correctly WITHOUT
referring back to the author for clarification.
```

这是 Category 3 最重要的 QA 创新：**用全新的 Claude 实例来验证文档的自解释性**。

设计动机（与 pptx 的 sub-agent Visual QA 类似）：写作者（主 Claude）对文档太熟悉，会无意中填补逻辑缺口。fresh sub-agent 没有这些背景，能真实模拟目标读者的阅读体验。

退出条件非常具体：sub-agent 能"正确回答问题"，而不是"文档看起来不错"。这是一个**行为验证**而非**内容审核**。

#### `doc-coauthoring` —— 迭代收敛检查

```markdown
Quality signal: After 3 iterations with no substantial changes,
ask user what can be removed.
```

这是一个**迭代收敛指标**：如果三轮迭代后变化越来越小，说明文档已趋于成熟，应该进入压缩精炼（而非继续添加）。

#### `internal-comms` —— 类型匹配验证（隐式）

```markdown
If the communication type doesn't match any example,
fall back to general-comms.md rather than guessing.
```

这是一个简单的分类准确性检查：没有匹配 → 显式回退，不静默地用错格式。

#### `doc-coauthoring` —— 最终人工验证点

```markdown
Final review: User verifies facts, links, and technical details
before completing.
```

质量责任的明确划分：Claude 验证文档的**结构和可读性**（通过 sub-agent），用户验证**事实准确性**（数据、链接、技术细节）。这避免了 Claude 对领域知识的过度自信。

**设计原理**：
- Sub-agent Reader Testing 是 Category 3 的核心 QA 创新：让 AI 扮演读者来验证文档的自解释性
- 退出条件（Reader 能正确回答问题）比内容评分（文档看起来好）更客观
- 明确划分 Claude 的质量责任（结构/可读性）和用户的质量责任（事实准确性）
- 收敛检查（3 轮后变化小 → 进入压缩）比无限迭代更有效率

---

### Technique 4: No External Tools Required（无需外部工具）

**设计思路**：Category 3 是所有分类中对外部工具**最少依赖**的，甚至少于 Category 2。

#### `internal-comms` —— 完全零依赖

```
internal-comms/
├── SKILL.md
└── examples/
    ├── 3p-updates.md
    ├── company-newsletter.md
    ├── faq-answers.md
    └── general-comms.md
```

没有 scripts/，没有外部库，没有 API key。Claude 读取 example 文件，生成文本输出。唯一的"工具"是 Claude 自身的写作能力。

#### `doc-coauthoring` —— 可选外部集成（非强制）

```markdown
Optional integrations (with connectors enabled):
- Slack: Share document for async review
- Teams: Collaborate in real-time
- Google Drive: Store final document
- SharePoint: Enterprise document management
```

这里的设计很有意思：外部集成**存在于 SKILL.md 中**，但标记为 "optional" 和 "with connectors enabled"。这意味着：
- 没有连接器时，skill 完全正常工作（输出为 artifact 或文件）
- 有连接器时，增加协作分发能力
- skill 的核心价值（协作写作工作流）不依赖任何外部服务

Sub-agent 是唯一的"工具"需求，而 sub-agent 是 Claude 自身的能力（没有时降级为用户手动测试）。

**对比 Category 1（docx/pptx/xlsx）**：
- Category 1 的工具依赖是**必要的**：LibreOffice 必须存在，validate.py 必须运行，否则功能缺失
- Category 3 的外部集成是**增强的**：没有也能完成核心任务，有则体验更好

**设计原理**：
- Enterprise & Communication 技能的核心价值在于**知识**（沟通框架、协作过程）而非工具
- 零工具依赖意味着最大可移植性：任何有 Claude 的环境都能运行这些 skills
- 可选集成（optional connectors）是正确的设计选择：不强制要求企业配置外部连接，但提供扩展路径

---

## 整体设计哲学：两个核心决策

### 决策 1：把"组织文化"和"协作智慧"内化进 SKILL.md

Category 1 内化了**领域技术规范**（Excel 色彩编码、XML 元素顺序）。Category 2 内化了**美学判断力**。Category 3 内化了**组织沟通智慧**：

| Skill | 内化了什么组织知识 |
|-------|-----------------|
| `internal-comms` | 公司内部通信的分类体系和格式约定（3P、newsletter 等） |
| `doc-coauthoring` | 协作写作的专家方法论（受众分析、渐进填充、Reader Testing） |

没有这些 skills，Claude 会用通用写作能力处理企业沟通，可能产出格式不符、语气偏差的结果。有了 skills，Claude 像一个懂公司文化的资深员工，知道向 CEO 汇报用什么格式，向团队更新用什么框架。

### 决策 2：Reader Testing —— 把质量验证交给"他者"

Category 3 最具洞察力的设计是 `doc-coauthoring` 的 Reader Testing：

```
主 Claude（写作者）→ 文档 → 新 Claude 实例（读者）→ 问答测试 → 迭代
```

这个设计承认了一个基本的认知限制：**写作者无法真正站在读者角度评估文档**，无论是人类作家还是 AI。解决方案不是让 Claude 更努力地自我检查，而是结构性地引入一个全新的视角。

这与 `pptx` 的 sub-agent Visual QA 是同一设计思路的不同应用：
- pptx：sub-agent 检查视觉问题（主 Claude 盯着代码，对布局变得麻木）
- doc-coauthoring：sub-agent 检查逻辑自洽性（主 Claude 熟悉背景，对逻辑缺口变得麻木）

**Category 3 的设计范式** = 把"组织沟通智慧"内化进 SKILL.md + 用"他者视角"（sub-agent 读者）解决写作者的确认偏误。
