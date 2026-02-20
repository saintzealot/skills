# Category 4 Skill 设计思路分析：Development & Technical

## 分析对象

README 分组中的 Development & Technical 示例：

| Skill | 文件 | 核心能力 |
|-------|------|---------|
| `mcp-builder` | `skills/mcp-builder/SKILL.md` | 创建高质量 MCP 服务器（TypeScript/Python） |
| `web-artifacts-builder` | `skills/web-artifacts-builder/SKILL.md` | React 应用 → 单 HTML artifact |
| `webapp-testing` | `skills/webapp-testing/SKILL.md` | Playwright 驱动的本地 Web 应用测试 |
| `skill-creator` | `skills/skill-creator/SKILL.md` | 创建 Agent Skills 的元技能 |

---

## 核心洞察：Development & Technical 的两个子模式

| 子模式 | 代表 | 特征 |
|--------|------|------|
| **构建型** | `mcp-builder` / `web-artifacts-builder` / `skill-creator` | 从无到有创建软件制品（MCP 服务器、React 应用、Skill 包） |
| **验证型** | `webapp-testing` | 对已有系统执行测试和检查 |

与其他分类的根本区别：
- Category 1（Document & Asset Creation）的输出是**文档文件**
- Category 2（Creative & Design）的输出是**视觉制品**
- Category 3（Enterprise & Communication）的输出是**文本沟通**
- Category 4（Development & Technical）的输出是**可运行的软件系统**

输出是可运行软件，这带来了不同的质量验证需求：不能只看输出格式是否正确，必须**实际运行并测试**。

---

## 四项 Key Techniques 的设计分析

---

### Technique 1: Embedded Style Guides and Brand Standards（嵌入式风格指南）

**设计思路**：Category 4 的"风格指南"是**工程规范**——命名约定、接口设计原则、反模式清单。内化的不是美学，而是软件工程的最佳实践。

#### `mcp-builder` —— Tool 命名规范 + Schema 设计规范

Tool 命名（最具体的工程规范之一）：

```markdown
Tool naming: use {service}_{action} format
Examples: github_create_issue, slack_send_message, drive_list_files
Rationale: LLMs understand tool purpose from name alone
```

Input Schema 规范：
```typescript
// REQUIRED: Zod schema with descriptions and constraints
const schema = z.object({
  repo: z.string().describe("Repository in 'owner/repo' format"),
  title: z.string().min(1).max(200).describe("Issue title")
});
```

Output 规范：
```markdown
Structured responses when available (structuredContent field)
Actionable error messages: "Repository 'foo/bar' not found.
Check that the repo exists and your token has access."
```

Annotation 规范：
```markdown
readOnlyHint: true    ← 不修改数据
destructiveHint: true ← 可能删除数据
idempotentHint: true  ← 重复调用安全
openWorldHint: false  ← 封闭世界假设
```

这些规范不是凭空发明的，而是 MCP 协议生态的最佳实践。Skill 把协议层面的工程判断力内化进来，让 Claude 像熟悉 MCP 规范的工程师一样设计 server。

#### `web-artifacts-builder` —— AI Slop 反模式清单（负向规范）

```markdown
Avoid common AI-generated patterns:
- Excessive centered layouts
- Purple gradients on white/dark backgrounds
- Uniform rounded corners (border-radius: 8px everywhere)
- Inter font as default
- Identical shadow values (box-shadow: 0 4px 6px rgba(0,0,0,0.1))
- Generic gradient hero sections
```

这是一个**反 AI 审美**的规范——与 `frontend-design` 的 NEVER 列表思路一致，Category 4 也用反向禁止来驱动差异化。

#### `webapp-testing` —— 测试策略决策树

```markdown
Decision tree:
1. Is it a static HTML file? → Read directly (no server needed)
2. Is it a dynamic webapp?
   → Is a server already running? → Use Playwright directly
   → No server running? → Use with_server.py helper
```

这是一个**操作前的诊断框架**，防止 Claude 对所有情况都用同一个方法（比如对静态 HTML 也启动 Playwright server，造成不必要的复杂度）。

#### `skill-creator` —— Skill 设计原则（最元层面的规范）

```markdown
Context window is a public good — only load what's needed.
Match degrees of freedom to task variability:
- High variability → high freedom (prose instructions)
- Low variability → low freedom (scripts)
Avoid: README.md, INSTALLATION_GUIDE.md, CHANGELOG.md
       (only operational content belongs in a skill)
```

"上下文窗口是公共资源"是一个工程伦理原则——设计 skill 时要考虑对其他 skills 和用户任务的影响。这不是美学，也不是格式规范，而是系统设计原则。

**设计原理**：
- Category 4 的风格指南是**工程规范**：命名约定、接口契约、系统设计原则
- 正向规范（Tool 命名格式、Schema 设计）和反向禁止（AI Slop 列表）都使用
- 最元层面的 skill（skill-creator）内化的是**设计 skill 的原则本身**

---

### Technique 2: Template Structures for Consistent Output（模板结构）

**设计思路**：Category 4 的模板是**软件工程工作流**，而非文档格式或认知框架。模板的作用是把复杂的技术流程分解为可重复执行的步骤序列。

#### `mcp-builder` —— 四阶段开发流程模板

```markdown
## Phase 1: Deep Research & Planning
- Read API documentation thoroughly
- Plan tool surface (what operations to expose)
- Design tool naming and schema
- Plan context management strategy

## Phase 2: Implementation
- Set up project structure
- Implement core infrastructure (transport, auth)
- Implement tools with full schemas
- Add error handling

## Phase 3: Review & Testing
- Code quality review
- Type coverage check
- MCP Inspector testing
- Fix identified issues

## Phase 4: Evaluation Creation
- Write 10 complex, realistic test questions in XML format
- Questions should test real-world usage scenarios
```

这是一个完整的软件开发生命周期模板，从需求（API 研究）到验证（Evaluation）。关键是每个阶段的**完成标准**是明确的（不是"写完代码"，而是"MCP Inspector 测试通过"）。

四阶段的最后一步（写 10 个评估问题）特别有设计感：它强制 Claude 在开发完成后系统性地思考"这个 server 能否处理真实的用户场景"。

#### `web-artifacts-builder` —— 脚手架脚本模板

```bash
# 初始化
bash scripts/init-artifact.sh <project-name>

# 开发（修改生成的文件）

# 打包
bash scripts/bundle-artifact.sh
# 输出：bundle.html（自包含的单文件 HTML artifact）
```

这是一个三步工作流模板，通过脚本完全封装了复杂的构建配置（Vite + Parcel + Tailwind + shadcn/ui）。Claude 的职责缩小为：实现业务逻辑（Step 2），其余由脚本处理。

预装了 40+ shadcn/ui 组件（通过 `shadcn-components.tar.gz`），避免每次都需要选择和配置 UI 组件库。

#### `webapp-testing` —— 侦察-然后-行动模板（Reconnaissance-Then-Action）

```markdown
## Pattern: Reconnaissance Before Action

Step 1: Navigate to target URL
Step 2: wait_for_load_state('networkidle')  ← CRITICAL WAIT
Step 3: Take screenshot → identify elements
Step 4: Inspect DOM → find selectors
Step 5: Execute automation actions
```

这个模板的核心洞察：**不能假设知道页面上有什么**，必须先"侦察"（截图 + DOM 检查），再行动。这防止了基于假设的 selector 选择，避免测试脚本因页面变化而失效。

`networkidle` 等待是强制的技术要求：在 SPA 应用中，`DOMContentLoaded` 之后页面可能仍在渲染，必须等网络请求稳定后再操作。

#### `skill-creator` —— 六步创建流程模板

```markdown
1. Understanding: Clarify what the skill needs to do
2. Planning: Design structure (SKILL.md + scripts/ + references/ + assets/)
3. Initialization: python scripts/init_skill.py <skill-name>
4. Editing: Write SKILL.md and add resources
5. Packaging: python scripts/package_skill.py <skill-folder>
6. Iteration: Test and refine based on feedback
```

这是一个**元级别**的工作流模板——创建 skill 的 skill 本身。值得注意的是第 3 步（脚手架生成）和第 5 步（打包验证）都有对应的脚本，减少 Claude 手工操作出错的可能。

**设计原理**：
- 软件开发类任务的模板是**阶段化的工作流**，每个阶段有明确的完成标准
- 脚手架脚本（init-artifact.sh, init_skill.py）是"模板的实例化"——把架构决策固化成可执行的代码
- 侦察-然后-行动模式是测试类任务的通用原则：不假设、先观察、再操作

---

### Technique 3: Quality Checklists Before Finalizing（最终确认前的质量检查）

**设计思路**：Category 4 的质量检查是所有分类中最系统化的——软件系统的质量可以通过运行代码来客观验证，不依赖美学判断或用户反馈。

#### `mcp-builder` —— MCP Inspector + 10 题评估体系

```markdown
## Phase 3 QA: Review & Testing

Code quality:
- Type coverage: all function parameters and returns typed
- Error handling: every tool has actionable error messages
- Async patterns: proper async/await usage

MCP Inspector testing:
- List tools → verify all expected tools present
- Call each tool with valid inputs → verify correct responses
- Test error cases → verify actionable error messages
- Test edge cases (empty lists, missing optional params)
```

```xml
<!-- Phase 4: 10-question Evaluation -->
<evaluation>
  <question>
    <description>Create an issue in repository X with specific labels</description>
    <expected_tools>github_create_issue, github_add_labels</expected_tools>
    <complexity>multi-step</complexity>
  </question>
  ...
</evaluation>
```

这是一个**两层验证**：
1. MCP Inspector 验证接口层（工具存在、格式正确）
2. 10 题 Evaluation 验证能力层（能否完成真实用户场景）

评估问题要求"complex, realistic"——不是单一 API 调用，而是涉及多步骤、多工具协作的真实场景。这迫使 Claude 在开发完成后思考 MCP server 的实际用途。

#### `webapp-testing` —— 截图确认作为测试前置步骤

```python
# CRITICAL: Never skip this step
page.wait_for_load_state('networkidle')
screenshot = page.screenshot()
# Inspect screenshot before writing selectors
```

截图检查不是最终 QA，而是**每个操作前的确认步骤**。这反转了常见的测试流程（先写 selector 再运行），把确认提前，减少"假设 selector 存在"导致的测试失败。

```markdown
After each fix, re-verify that the fix worked.
Use console_logging.py to capture browser logs for debugging.
```

#### `skill-creator` —— 三级验证

```bash
# Quick check
python scripts/quick_validate.py <skill-folder>

# Full packaging + validation
python scripts/package_skill.py <skill-folder>
```

quick_validate.py 是开发中的快速检查，package_skill.py 是正式打包前的完整验证。两级工具对应不同的使用场景（开发中频繁检查 vs 发布前全面验证）。

#### `web-artifacts-builder` —— 功能检查

```markdown
## After bundling

1. Open bundle.html in browser
2. Verify all components render correctly
3. Test all interactive features (state management, routing)
4. Check console for errors
```

这是最基础的 QA：在真实浏览器中运行，确认没有运行时错误。对 AI Slop 的禁止也需要视觉检查（看是否无意中使用了禁止的模式）。

**设计原理**：
- 软件系统的质量验证必须通过**实际运行**来确认（不能只看代码）
- 两层验证（接口层 + 能力层）比单一验证更全面
- 截图前置（侦察）比操作后截图（确认）更能防止操作失败
- 快速检查 + 完整验证的双级工具设计，在开发效率和发布质量之间取得平衡

---

### Technique 4: No External Tools Required（无需外部工具）

**设计思路**：Category 4 是外部工具依赖**最复杂**的分类——开发类任务本质上需要工具链。关键问题是：哪些工具随 skill 打包，哪些工具假设已在环境中存在？

#### `webapp-testing` —— 最轻量：仅一个 Helper 脚本

```
webapp-testing/
├── SKILL.md
├── scripts/
│   └── with_server.py   ← 服务器生命周期管理
└── examples/
    ├── console_logging.py
    ├── element_discovery.py
    └── static_html_automation.py
```

Playwright 本身作为系统依赖（假设已安装）。Skill 只打包了：
- `with_server.py`：封装服务器启动/停止的 helper（这是繁琐且易错的部分）
- `examples/`：Playwright 代码模式的参考

设计选择：Playwright 是标准工具，不需要随 skill 重新分发；但服务器生命周期管理是容易出错的环节，值得封装成 helper。

#### `mcp-builder` —— 最重：参考文档 + 评估脚本

```
mcp-builder/
├── SKILL.md
├── scripts/
│   ├── connections.py        ← API 连接模式
│   ├── evaluation.py         ← 评估运行器
│   └── example_evaluation.xml ← XML 格式参考
└── reference/
    ├── mcp_best_practices.md
    ├── node_mcp_server.md    ← TypeScript 模式和示例
    ├── python_mcp_server.md  ← Python 模式和示例
    └── evaluation.md         ← 评估创建指南
```

TypeScript/Python SDK 和 MCP Inspector 是外部工具（假设可安装）。Skill 打包了：
- 四个参考文档：把 MCP 最佳实践编码进 skill，避免 Claude 每次都需要查阅外部文档
- evaluation.py + XML：把评估能力封装进 skill（评估是 MCP server 开发的特殊需求，不是通用工具）

#### `skill-creator` —— 最特殊：打包自身工具链

```
skill-creator/
├── scripts/
│   ├── init_skill.py    ← 生成 skill 模板
│   ├── package_skill.py ← 验证并打包 .skill 文件
│   └── quick_validate.py ← 快速验证
└── references/
    ├── output-patterns.md
    └── workflows.md
```

skill-creator 打包的是**创建 skill 所需的工具**。这是一个自包含的开发环境：不依赖任何外部 skill 开发工具，因为这个 skill 本身就提供了全套工具。

#### `web-artifacts-builder` —— 最大的捆绑资产

```
web-artifacts-builder/
├── scripts/
│   ├── init-artifact.sh           ← 初始化 React 项目
│   ├── bundle-artifact.sh         ← 打包为单 HTML
│   └── shadcn-components.tar.gz  ← 40+ 预装组件 (!!)
```

`shadcn-components.tar.gz` 是最显著的捆绑资产：把 40+ shadcn/ui 组件预先打包，避免每次构建都需要重新安装和配置。Node.js/npm/Vite/Parcel 是系统依赖，但构建配置（Parcel 配置、Tailwind 配置）通过 init 脚本自动化。

**工具依赖谱系对比**：

| Skill | 系统依赖（假设存在） | Skill 打包的工具 | 无需任何工具 |
|-------|-------------------|----------------|-------------|
| `webapp-testing` | Playwright | with_server.py, examples | — |
| `mcp-builder` | TypeScript/Python SDK | reference docs, evaluation.py | — |
| `skill-creator` | Python | init_skill.py, package_skill.py | — |
| `web-artifacts-builder` | Node.js, npm | init/bundle scripts, shadcn tarball | — |

**设计原理**：
- 开发类任务无法完全消除工具依赖，关键是区分"通用工具（系统假设）"和"专属工具（skill 打包）"
- 最值得打包的工具：反复执行且易出错的操作（服务器生命周期、打包流程）
- 最不需要打包的工具：广泛存在的通用框架（Playwright、Node.js、Python）
- `shadcn-components.tar.gz` 体现了"预配置胜于再配置"：把成熟的技术选型固化进 skill，避免每次都重复决策

---

## 整体设计哲学：两个核心决策

### 决策 1：把"工程判断力"和"技术规范"内化进 SKILL.md

Category 4 内化的是资深工程师的工程判断：

| Skill | 内化了什么工程知识 |
|-------|-----------------|
| `mcp-builder` | MCP 协议设计最佳实践（命名、schema、error messages、annotations） |
| `web-artifacts-builder` | React 应用架构决策（Vite+Parcel+shadcn/ui）+ AI Slop 反模式 |
| `webapp-testing` | 测试策略决策树 + 侦察-然后-行动的测试哲学 |
| `skill-creator` | 如何设计高质量 Skill（渐进式披露、上下文效率、自由度匹配） |

没有这些 skills，Claude 处理开发类任务时会：
- 随意命名 MCP tools（tool1, createIssue, etc.）
- 直接操作 DOM 而不先截图侦察
- 设计过度复杂或过度简单的 Skill 结构

有了这些 skills，Claude 像有 MCP 开发经验、熟悉测试最佳实践、懂得如何设计清晰 API 的工程师一样工作。

### 决策 2：评估驱动的完成标准

Category 4 最重要的设计是**对"完成"有更高标准**：

- Category 1（Document & Asset Creation）：validate.py 通过 = 完成
- Category 3（Enterprise & Communication）：Reader 能正确回答问题 = 完成
- Category 4（Development & Technical）：

```
webapp-testing: 在实际浏览器中测试通过 = 完成
mcp-builder: MCP Inspector 测试 + 10 题评估 = 完成
web-artifacts-builder: 在浏览器中实际渲染并交互 = 完成
skill-creator: package_skill.py 通过 + 实际触发测试 = 完成
```

软件系统的质量无法通过静态分析完全验证，必须**运行代码**。这个原则被明确编码进了每个 skill 的完成标准里。

特别值得注意的是 `mcp-builder` 的 10 题评估体系——它不仅要求 MCP server 能运行，还要求 Claude 证明这个 server 能完成真实用户场景。这是一种**从验收测试定义完成标准**的工程文化，内化进了 skill 的工作流。

**Category 4 的设计范式** = 把"工程判断力和技术规范"内化进 SKILL.md + 用"运行验证而非静态检查"作为完成标准。
