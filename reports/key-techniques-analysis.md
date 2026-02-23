# Agent Skills 三大类别 Key Techniques 实现分析报告

> 来源依据：[The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
> 分析对象：`anthropics/skills` 仓库中的生产级 skill 样例

---

## 一、Document & Asset Creation（文档与资产创作）

### Key Techniques

| # | 技术特征 |
|---|---------|
| 1 | Embedded style guides and brand standards（嵌入式风格指南与品牌标准）|
| 2 | Template structures for consistent output（一致性输出的模板结构）|
| 3 | Quality checklists before finalizing（完成前质量检查清单）|
| 4 | No external tools required（无需外部工具，使用 Claude 内置能力）|

---

### 典型样例 A：`frontend-design`

**选取理由**：该 skill 在纯文本指令中完整实现了 4 项 key techniques，是最简洁、最纯粹的 Category 1 代表。

#### Technique 1 — Embedded Style Guides and Brand Standards

`SKILL.md` 的 `Frontend Aesthetics Guidelines` 一节（第 27–38 行）将风格指南**直接硬编码进 skill body**：

```
- Typography: Choose fonts that are beautiful, unique, and interesting.
  Avoid generic fonts like Arial and Inter...
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for
  consistency. Dominant colors with sharp accents outperform timid,
  evenly-distributed palettes.
- Motion: Use animations for effects and micro-interactions...
- Spatial Composition: Unexpected layouts. Asymmetry. Overlap...
- Backgrounds & Visual Details: Add contextual effects and textures...
```

关键在于品牌**禁止条款**（第 36 行）：

```
NEVER use generic AI-generated aesthetics like overused font families
(Inter, Roboto, Arial, system fonts), cliched color schemes
(particularly purple gradients on white backgrounds)...
```

这是一套完整的品牌排除标准，凡是生成的前端界面都必须通过该标准的负向筛选。这种"白名单原则 + 黑名单条款"的组合，等同于企业 Brand Guidelines 手册中常见的 DO/DON'T 规范，但直接嵌入 skill，无需 Claude 从外部检索。

#### Technique 2 — Template Structures for Consistent Output

`Design Thinking` 一节（第 12–19 行）定义了每次生成时必须完成的**结构性思考框架**：

```
Before coding, understand the context and commit to a BOLD aesthetic:
- Purpose: What problem does this interface solve? Who uses it?
- Tone: Pick an extreme: brutally minimal, maximalist chaos, retro-
  futuristic, organic/natural, luxury/refined, playful/toy-like...
- Constraints: Technical requirements (framework, performance, accessibility)
- Differentiation: What makes this UNFORGETTABLE?
```

这是一个输出前的**思维模板（cognitive template）**：无论用户要求构建 landing page、dashboard 还是 React 组件，Claude 都必须先经过这四步结构化思考，再动手写代码。这保证了输出格式的一致性，而非内容的一致性——每个界面视觉不同，但思维路径相同。

#### Technique 3 — Quality Checklists Before Finalizing

第 40–41 行提供了完成标准的质量判据：

```
Match implementation complexity to the aesthetic vision. Maximalist
designs need elaborate code with extensive animations and effects.
Minimalist or refined designs need restraint, precision, and careful
attention to spacing, typography, and subtle details.
```

以及第 21–25 行定义了最终输出必须同时满足的四项标准：

```
Then implement working code that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail
```

这四条是**隐式 checklist**：Claude 在输出代码之前必须满足全部四项，否则输出不符合 skill 要求。

#### Technique 4 — No External Tools Required

整个 skill 不调用任何 MCP server、不执行任何脚本、不读取外部文件。风格知识、审美判断、代码生成全部由 Claude 本身完成。这使 skill 在任意 Claude 部署环境中（web、API、Claude Code）**零配置可运行**。

---

### 典型样例 B：`xlsx`（Quality Checklist 的工程级实现）

**选取理由**：`xlsx` 将 Technique 3（质量检查清单）提升到了工程验证的层次，是所有 skill 中该技术实现最深入的案例。

#### Technique 3 的工程级实现

**第一层——规范性 Checklist（第 9–64 行）**：

`Requirements for Outputs` 一节定义了三级质量标准：

```
All Excel files:
  - Professional Font (Arial / Times New Roman)
  - Zero Formula Errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)
  - Preserve Existing Templates

Financial models:
  - Color Coding Standards (Blue=inputs, Black=formulas,
    Green=cross-sheet links, Red=external links, Yellow=key assumptions)
  - Number Formatting Standards (currency, zeros, percentages, multiples)
  - Formula Construction Rules (assumptions in separate cells, no
    hardcoded values, documentation for hardcodes)
```

**第二层——程序化 Checklist（第 228–263 行）**：

```
Formula Verification Checklist:
  Essential Verification:
  - [ ] Test 2-3 sample references
  - [ ] Column mapping: Confirm Excel columns match
  - [ ] Row offset: Remember Excel rows are 1-indexed

  Common Pitfalls:
  - [ ] NaN handling
  - [ ] Far-right columns (FY data often in columns 50+)
  - [ ] Division by zero (#DIV/0!)
  - [ ] Wrong references (#REF!)
  - [ ] Cross-sheet references format

  Formula Testing Strategy:
  - [ ] Start small: Test on 2-3 cells before applying broadly
  - [ ] Verify dependencies
  - [ ] Test edge cases: zero, negative, very large values
```

**第三层——自动化验证（第 132–147 行）**：

```
5. Recalculate formulas (MANDATORY IF USING FORMULAS):
   python scripts/recalc.py output.xlsx

6. Verify and fix any errors:
   - If status is errors_found, check error_summary for specific
     error types and locations
   - Fix the identified errors and recalculate again
```

这三层形成了 **规范定义 → 人工核查 → 机器验证** 的完整质量闭环，确保任何输出都经过了多重核查。

#### Technique 1 — Embedded Brand Standards

颜色编码规范（第 26–33 行）是金融行业内部事实标准（industry-standard color conventions），直接嵌入 skill：

```
Blue text (RGB: 0,0,255): Hardcoded inputs
Black text (RGB: 0,0,0): ALL formulas and calculations
Green text (RGB: 0,128,0): Links from other worksheets
Red text (RGB: 255,0,0): External links to other files
Yellow background (RGB: 255,255,0): Key assumptions needing attention
```

用户不需要告诉 Claude"按行业惯例着色"，skill 已将行业标准内化。

---

### 典型样例 C：`theme-factory`（Template Structure 的显式实现）

**选取理由**：`theme-factory` 最直接地展示了 Technique 2 的设计意图——将"一致性输出"通过预定义模板包而非即兴生成实现。

#### Technique 2 — Template Structures for Consistent Output

该 skill 提供 10 套预定义主题文件（`themes/` 目录），每套包含完整规格：

```
1. Ocean Depths    6. Arctic Frost
2. Sunset Boulevard  7. Desert Rose
3. Forest Canopy    8. Tech Innovation
4. Modern Minimalist  9. Botanical Garden
5. Golden Hour     10. Midnight Galaxy
```

应用流程（第 50–58 行）是强制顺序：

```
Application Process:
1. Read the corresponding theme file from themes/ directory
2. Apply the specified colors and fonts consistently throughout
3. Ensure proper contrast and readability
4. Maintain the theme's visual identity across all slides
```

这里的关键设计是：Claude **不生成**主题，而是**读取**主题文件后**应用**。这将"创意可变性"与"风格一致性"分离——主题的创意决策在 skill 制作时已固化，运行时只做忠实执行。

用户交互流程（第 20–27 行）也被模板化：

```
1. Show the theme-showcase.pdf file (do not modify)
2. Ask for their choice
3. Wait for selection (explicit confirmation)
4. Apply the theme
```

这四步强制用户确认，防止 Claude 自行猜测用户偏好，是 Technique 2 中"一致性"的行为层保障。

---

## 二、Workflow Automation（工作流自动化）

### Key Techniques

| # | 技术特征 |
|---|---------|
| 1 | Step-by-step workflow with validation gates（带验证门控的分步工作流）|
| 2 | Templates for common structures（通用结构模板）|
| 3 | Built-in review and improvement suggestions（内置审阅与改进建议）|
| 4 | Iterative refinement loops（迭代精炼循环）|

---

### 典型样例：`doc-coauthoring`

**选取理由**：该 skill 在一个统一的文档协作场景中完整实现了全部 4 项 key techniques，且实现方式相互嵌套、层次分明，是 Category 2 最具教学价值的样例。

#### Technique 1 — Step-by-step Workflow with Validation Gates

整个 skill 被设计为三阶段顺序工作流，每个阶段有明确的**退出条件（validation gate）**：

**Stage 1 → Stage 2 的验证门（第 96–102 行）**：

```
Exit condition:
Sufficient context has been gathered when questions show understanding
— when edge cases and trade-offs can be asked about without needing
basics explained.

Transition: Ask if there's any more context they want to provide at
this stage, or if it's time to move on to drafting the document.
```

Claude 不能自行判断"我已经够了解了"就跳入下一阶段，必须通过上述条件测试：能否独立提出 edge case 级别的问题？这是一个**能力验证门**。

**Stage 2 → Stage 3 的验证门（第 231–239 行）**：

```
When all sections are drafted and refined:
Announce all sections are drafted. Indicate intention to review
the complete document one more time.

Review for overall coherence, flow, completeness.
Provide any final suggestions.
Ask if ready to move to Reader Testing, or want to refine anything.
```

进入 Stage 3 之前，Claude 必须完成全文整体审阅并获得用户明确授权，不能单方面推进。

**Stage 2 内部每个 Section 的验证门（第 219 行）**：

```
When section is done, confirm [SECTION NAME] is complete.
Ask if ready to move to the next section.
```

这形成了三级嵌套验证门结构：**全局阶段级 → 文档阶段级 → section 级**。

#### Technique 2 — Templates for Common Structures

**文档结构模板（第 122–143 行）**：

```
If user doesn't know what sections they need:
  Based on the type of document and template, suggest 3-5 sections
  appropriate for the doc type.

Once structure is agreed:
  Create the initial document structure with placeholder text for
  all sections.

  Create artifact with all section headers and brief placeholder
  text like "[To be written]" or "[Content here]".
```

Claude 为不知道如何组织文档的用户提供**推荐结构模板**，并立即生成带占位符的完整骨架。这个"先建骨架，再填内容"的模式使协作双方始终有一个共同的可见工作对象。

**初始问题模板（第 33–39 行）**：

```
Start by asking the user for meta-context:
1. What type of document is this?
2. Who's the primary audience?
3. What's the desired impact when someone reads this?
4. Is there a template or specific format to follow?
5. Any other constraints or context to know?
```

这 5 个问题是**通用情境收集模板**，无论用户在写 PRD、RFC 还是 Decision Doc，第一步的信息收集结构完全相同。

#### Technique 3 — Built-in Review and Improvement Suggestions

**Stage 3 Reader Testing** 是最具创意的内置审阅机制（第 242–331 行）：

```
Goal: Test the document with a fresh Claude (no context bleed)
to verify it works for readers.

If access to sub-agents is available (e.g., in Claude Code):
  Step 1: Predict Reader Questions (5-10 realistic questions)
  Step 2: Test with Sub-Agent (invoke fresh Claude with just the
          document and each question)
  Step 3: Run Additional Checks (ambiguity, false assumptions,
          contradictions)
  Step 4: Report and Fix
```

该机制的核心洞察是：**作者和读者拥有不同的信息量**。通过启动一个没有对话上下文的 sub-agent 来扮演"冷读者"，检测文档中的隐性假设和表达歧义。这是一种无法由作者自身完成的审阅视角，skill 通过智能体架构将其内置为流程的第三阶段。

**近完成时的整体审阅（第 224–231 行）**：

```
As approaching completion (80%+ of sections done):
Re-read the entire document and check for:
- Flow and consistency across sections
- Redundancy or contradictions
- Anything that feels like "slop" or generic filler
- Whether every sentence carries weight
```

这是内置于 skill 的**文档健康检查清单**，Claude 在用户未要求的情况下主动执行。

#### Technique 4 — Iterative Refinement Loops

每个 section 的处理被设计为 6 步循环（第 153–219 行）：

```
Step 1: Clarifying Questions (5-10 questions about section content)
Step 2: Brainstorming (5-20 candidate items)
Step 3: Curation (user selects what to keep/remove/combine)
Step 4: Gap Check (anything important missing?)
Step 5: Drafting (generate actual section content)
Step 6: Iterative Refinement (str_replace edits until satisfied)
```

第 6 步的终止机制（第 215–218 行）：

```
After 3 consecutive iterations with no substantial changes,
ask if anything can be removed without losing important information.
```

这是一个**收敛判断机制**：连续 3 轮无实质变化 → 问是否进入精简优化阶段。这防止了无效循环，同时保证充分迭代。

循环中的关键约束（第 206–211 行）：

```
As user provides feedback:
- Use str_replace to make edits (never reprint the whole doc)
- If user edits doc directly: mentally note the changes and keep them
  in mind for future sections (shows their preferences)
```

`str_replace` 约束保证了迭代精炼的**外科手术精确性**，避免每次迭代都重写全文导致上下文污染。

---

### 辅助案例：`skill-creator`（Validation Gate 的工程级实现）

`skill-creator` 本身是一个 Category 2 skill，其核心是带验证门控的 6 步工作流（第 203–358 行）：

```
Step 1: Understand the skill with concrete examples
Step 2: Plan reusable skill contents
Step 3: Initialize (run init_skill.py)            ← 工具执行验证
Step 4: Edit the skill
Step 5: Package the skill (run package_skill.py)  ← 自动验证门
Step 6: Iterate based on real usage
```

**Step 5 的验证门**（第 321–344 行）最能体现 Technique 1：

```
The packaging script will:
1. Validate the skill automatically, checking:
   - YAML frontmatter format and required fields
   - Skill naming conventions and directory structure
   - Description completeness and quality
   - File organization and resource references

2. Package the skill if validation passes

If validation fails, the script reports errors and exits without
creating a package. Fix any validation errors and run again.
```

这是**程序化强制门控**：不通过验证，流程物理上无法推进（没有 `.skill` 输出文件）。这比文字说"请检查一下"强得多——它将质量标准编码进了工具链。

---

## 三、MCP Enhancement（MCP 增强）

### Key Techniques

| # | 技术特征 |
|---|---------|
| 1 | Coordinates multiple MCP calls in sequence（按序协调多个 MCP 调用）|
| 2 | Embeds domain expertise（嵌入领域专业知识）|
| 3 | Provides context users would otherwise need to specify（提供用户本需手动指定的上下文）|
| 4 | Error handling for common MCP issues（常见 MCP 问题的错误处理）|

---

### 典型样例：`webapp-testing`

**选取理由**：该 skill 聚焦于 Playwright 这一具体工具栈，将"如何与本地 web 应用交互"这个需要大量专业判断的任务，通过 4 项 key techniques 转化为可重复执行的规程。

#### Technique 1 — Coordinates Multiple MCP Calls in Sequence

`webapp-testing` 的核心是一棵**决策树**（第 18–33 行），它协调了多步骤的工具调用序列：

```
User task → Is it static HTML?
  ├─ Yes → Read HTML file directly to identify selectors
  │         ├─ Success → Write Playwright script using selectors
  │         └─ Fails/Incomplete → Treat as dynamic (below)
  │
  └─ No (dynamic webapp) → Is the server already running?
      ├─ No → Run: python scripts/with_server.py --help
      │        Then use the helper + write simplified Playwright script
      │
      └─ Yes → Reconnaissance-then-action:
          1. Navigate and wait for networkidle
          2. Take screenshot or inspect DOM
          3. Identify selectors from rendered state
          4. Execute actions with discovered selectors
```

这棵决策树定义了工具调用的**条件化序列**：先 Read HTML（或 navigate），再 screenshot/DOM inspect，再提取 selectors，最后执行 actions。每一步的结果决定下一步使用哪个工具。这正是"协调多个 MCP 调用"的本质——不是随机调用，而是有序编排。

多服务器场景（第 43–50 行）展示了更复杂的调用协调：

```
Multiple servers (e.g., backend + frontend):
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

`with_server.py` 作为**服务生命周期管理器**，在 Playwright 脚本执行前确保所有依赖服务就绪，实现了跨服务调用的编排。

#### Technique 2 — Embeds Domain Expertise

**Reconnaissance-Then-Action Pattern**（第 65–81 行）是该 skill 嵌入的核心专业知识：

```
1. Inspect rendered DOM:
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()

2. Identify selectors from inspection results

3. Execute actions using discovered selectors
```

这个模式体现了 Playwright 专家的一个关键洞察：**不要假设你知道 DOM 结构，先观察再行动**。对于动态渲染的 web 应用，静态分析源码无法可靠获取运行时 selectors。这是只有具备 Playwright 实战经验的工程师才会掌握的反直觉知识，skill 将其显式编码。

第 13–14 行关于 `with_server.py` 的使用规范也体现了领域专业知识：

```
Always run scripts with --help first to see usage. DO NOT read the
source until you try running the script first and find that a
customized solution is absolutely necessary. These scripts can be
very large and thus pollute your context window.
```

这条"先 `--help`，再考虑读源码"的规则是 **token 经济学专业知识**——它告诉 Claude 如何在能力获取和上下文消耗之间做权衡。普通用户不会意识到这个问题，skill 直接将最优策略内化。

#### Technique 3 — Provides Context Users Would Otherwise Need to Specify

第 52–63 行的 Playwright 脚本模板预置了所有关键配置：

```python
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # Always launch chromium
                                                 # in headless mode
    page = browser.new_page()
    page.goto('http://localhost:5173')
    page.wait_for_load_state('networkidle')      # CRITICAL: Wait for JS
    # ... your automation logic
    browser.close()
```

三个隐性决策被 skill 代为做出：

1. **浏览器选择**：`chromium`（而非 firefox 或 webkit）
2. **headless 模式**：`headless=True`（服务器环境无 GUI）
3. **等待策略**：`networkidle`（而非 `load` 或 `domcontentloaded`）

用户在提需求时只会说"测试我的 web 应用"，不会说"用 chromium headless 模式等待 networkidle 后再交互"。skill 通过这些预置选项将专业配置知识内化，用户无需知道这些选项的存在。

`examples/` 目录（第 93–97 行）进一步预提供了常用场景的完整上下文：

```
Reference Files:
- examples/element_discovery.py     - Discovering buttons, links, inputs
- examples/static_html_automation.py - Using file:// URLs for local HTML
- examples/console_logging.py       - Capturing console logs
```

这些示例文件是**可复用的上下文供给**，Claude 可以按需读取，而不需要用户在每次会话中重新描述需求模式。

#### Technique 4 — Error Handling for Common MCP Issues

**Common Pitfall 部分**（第 79–82 行）直接针对 Playwright 最高频的错误场景：

```
Common Pitfall:
❌ Don't inspect the DOM before waiting for networkidle on dynamic apps
✅ Do wait for page.wait_for_load_state('networkidle') before inspection
```

这一条覆盖了绝大多数 Playwright 初始化问题的根因：**过早交互**。skill 将这个"错误模式 → 正确模式"的对比直接内嵌，使 Claude 在生成代码时优先遵循正确模式。

`Best Practices` 一节（第 84–90 行）进一步列出了防御性编程规范：

```
- Use sync_playwright() for synchronous scripts
- Always close the browser when done
- Use descriptive selectors: text=, role=, CSS selectors, or IDs
- Add appropriate waits: page.wait_for_selector() or
  page.wait_for_timeout()
```

`browser.close()` 的强制要求防止了资源泄漏，`descriptive selectors` 的优先级顺序（`text=` > `role=` > CSS > ID）内嵌了 Playwright 的可维护性最佳实践。这些规范无需用户每次在 prompt 中指定，skill 已将其设为默认行为。

---

### 辅助案例：`mcp-builder`（Embeds Domain Expertise 的深度实现）

`mcp-builder` 聚焦于 Technique 2 的深度，在 4 个阶段中系统化地嵌入了 MCP 协议设计的领域专业知识。

**工具命名规范**（第 27–30 行）：

```
Tool Naming and Discoverability:
Clear, descriptive tool names help agents find the right tools.
Use consistent prefixes (e.g., github_create_issue, github_list_repos)
and action-oriented naming.
```

**工具标注系统**（第 119–124 行）——这是 MCP 协议的高级特性，普通开发者难以发现：

```
Annotations:
- readOnlyHint: true/false
- destructiveHint: true/false
- idempotentHint: true/false
- openWorldHint: true/false
```

**分层 reference 文件架构**（第 197–236 行）将领域知识按需加载：

```
Core MCP Documentation (Load First)
SDK Documentation (Load During Phase 1/2)
Language-Specific Implementation Guides (Load During Phase 2)
Evaluation Guide (Load During Phase 4)
```

每类文档只在对应阶段才加载进上下文，这是 Technique 2 与 Progressive Disclosure 原则的结合——**领域知识也需要分层供给**，而非一次性全部注入。

---

## 总结：三类别 Key Techniques 实现对比

| Key Technique | Category 1 实现方式 | Category 2 实现方式 | Category 3 实现方式 |
|---|---|---|---|
| **知识嵌入方式** | 风格规则直写 SKILL.md body | 工作流程序写入 SKILL.md | 领域判断规则写入决策树 |
| **结构化手段** | DO/DON'T 规范 + Checklist | 阶段 + 步骤 + 退出条件 | 决策树 + 顺序脚本调用 |
| **质量保障** | 隐式 checklist + 自动验证脚本 | 验证门控 + sub-agent 测试 | 反模式警告 + 防御性代码模板 |
| **用户认知负担** | 无需了解风格细节 | 无需知道最优工作流顺序 | 无需知道工具配置参数 |
| **典型 skill** | `frontend-design` / `xlsx` / `theme-factory` | `doc-coauthoring` / `skill-creator` | `webapp-testing` / `mcp-builder` |

三类别的核心差异在于**知识的性质**不同：

- **Category 1** 嵌入的是"产出应该长什么样"的**规范性知识**
- **Category 2** 嵌入的是"过程应该怎么走"的**程序性知识**
- **Category 3** 嵌入的是"工具应该如何用"的**操作性知识**

Skills 通过统一的 SKILL.md 格式，将这三类截然不同的专业知识转化为 Claude 可直接执行的指令，是其作为"AI 专业化机制"最核心的价值所在。
