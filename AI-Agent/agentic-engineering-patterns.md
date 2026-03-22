# Agentic Engineering Patterns

> 来源：[Simon Willison - Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/)
> 学习日期：2026-03-22

## 概述

Simon Willison（Django 联合创始人）的实战指南，系统总结与 AI 编码 Agent（Claude Code、OpenAI Codex、Gemini CLI）高效协作的模式与原则。共 6 大部分、15 章节，拆为 5 个学习单元。

---

## 单元 1：核心认知转变

### 1.1 什么是 Agentic Engineering

- **Agent 本质**：在循环中调用工具来达成目标的软件（LLM ←→ 工具调用 ←→ 结果反馈 ←→ 再推理）
- **Agentic Engineering**：在编码 Agent 辅助下进行软件开发的实践
- **与 Vibe Coding 的区别**：Vibe Coding 不审查产出做原型；Agentic Engineering 要求你把代码带到生产级别
- **人的核心价值**：决定"写什么代码"——每个问题有多种解法和不同权衡，这部分判断力 Agent 替代不了

### 1.2 Writing Code is Cheap Now

- **核心认知转变**：代码生成成本趋近于零，还能并行跑多个 Agent
- **关键区分**：生成代码便宜了，但交付好代码依然昂贵
- 好代码标准：能工作且你确认过、解决正确的问题、有测试保护、简单最小化、不过度工程
- **行为改变**：重新评估以前"不值得花时间"的功能，让 Agent 异步跑一下成本极低

---

## 单元 2：工程师的新角色

### 2.1 Hoard Things You Know How to Do

- **核心**：每个技巧只需搞懂一次，记下来，Agent 可以无限复用
- 方法：博客、TIL、GitHub 仓库、单页工具——形式不重要，关键是有可工作的代码示例
- **组合策略**：告诉 Agent 结合两个已有方案构建新东西（案例：Tesseract.js + PDF.js → PDF OCR 工具）
- **本质**：知识的复利效应，技能库越大 × Agent 执行力 = 指数级产出

### 2.2 AI Should Help Us Produce Better Code

用 AI 写出更差的代码是选择，不是必然：
- **消除技术债**：重构任务是 Agent 完美场景（改名、拆文件、合并重复），异步后台跑
- **扩展方案空间**：Agent 做探索性原型成本极低，可以评估更多技术方案
- **复合工程循环**：复盘记录有效做法 → 写入 Agent 指令 → 小改进复利叠加

### 2.3 Anti-patterns（反模式）

**头号反模式：提交自己没 review 过的 PR**
- 好的 Agentic PR：代码能工作且你有信心、变更足够小、有上下文说明、PR 描述自己审查过
- **新职业素养**：附上工作证据（测试截图/视频/实现选择说明），证明你审查过

---

## 单元 3：Agent 工作原理

### 3.1 How Coding Agents Work

**Agent = LLM + System Prompt + Tools → 循环运行**

四个核心组件：
- **LLM**：token 补全器，按 token 计费；多模态模型图片也转成 token 处理
- **Chat 模板**：LLM 无记忆，每次重放完整对话历史（成本随对话增长）；Token 缓存可降本
- **工具调用**：Agent 与普通聊天的根本区别；编码 Agent 最强工具是 Bash/代码执行，形成写代码→跑验证的闭环
- **System Prompt**：隐藏的数百行指令，定义行为和工具；不同 Agent 产品的核心差异化
- **Reasoning**：2025 年进步，模型先思考再回答，对复杂调试特别有效

### 3.2 Using Git with Coding Agents

Git 是安全网——所有改动可追踪、可回滚。关键是知道 Git 能做什么，用自然语言指挥 Agent：
- `Review changes made today` — 快速加载项目上下文
- `Sort out this git mess` — 解决合并冲突
- `Find and recover my code that does ...` — 搜索 reflog/stash 找回代码
- `Use git bisect to find when this bug was introduced` — 自动化二分查找

**Git 历史观**：不是永久记录，而是精心编写的叙事故事。重写历史是编辑叙事，Agent 擅长。

### 3.3 Subagents

解决 context window 有限的核心策略：
- **原理**：派出独立 context 的 Agent 副本，完成任务后返回精简结果，保护主 context
- **三种模式**：顺序（简单）、并行（更快，可用便宜模型）、专业化（代码审查/测试/调试）
- **注意**：不要过度专业化，核心价值是管理 context，不是搞微服务架构

---

## 单元 4：测试体系

三层测试防线：

### 4.1 Red/Green TDD

- **提示词**：`Use red/green TDD`（六个字编码完整工程纪律）
- 防两个风险：代码不工作 + 多写不需要的代码
- **"Red" 的意义**：没见过测试失败，就不能确定测试真的在测你想测的东西
- 流程：写测试 → 确认失败（红）→ 写实现 → 确认通过（绿）

### 4.2 First Run the Tests

- **提示词**：`First run the tests`（每次用 Agent 打开已有项目时用）
- 三重效果：Agent 发现测试存在、理解项目规模、建立测试意识
- 本质：用极短提示词激活模型内置的工程纪律

### 4.3 Agentic Manual Testing

- **核心论点**：通过测试 ≠ 按预期工作，发布前要用自己的眼睛看到功能在工作
- 测试手段：`python -c` 快速验证、curl 测 API、Playwright 浏览器自动化 + 截图视觉验证
- **防伪造**：Showboat 的 `exec` 命令记录实际命令和真实输出，杜绝 Agent 编造测试结果
- **闭环**：手动测试发现的问题补充到自动化测试中

| 层级 | 方法 | 防什么 |
|------|------|--------|
| 第一层 | Red/Green TDD | 代码不工作、多写不需要的代码 |
| 第二层 | First Run the Tests | Agent 缺乏测试意识 |
| 第三层 | 手动测试 | 通过测试但不符预期、Agent 伪造结果 |

---

## 单元 5：理解与实战

### 5.1 认知债务（Cognitive Debt）

- **定义**：当我们不再理解 Agent 写的代码如何工作时，积累的心智负担
- 简单代码可以不深究，但**复杂应用逻辑必须理解**——否则无法自信推理未来的功能变更
- 两种还债方法：线性走读 → 交互式解释（递进）

### 5.2 Linear Walkthrough（线性代码走读）

- **场景**：快速开发后发现自己不理解自己的代码
- **方法**：让 Agent 生成结构化的逐文件走读文档
- **关键技巧**：用 `sed/grep/cat` 提取代码片段，而非让模型手动复制——防止幻觉
- **工具**：Showboat（`showboat note` 写注释，`showboat exec` 执行命令记录真实输出）
- **启示**：AI 辅助开发不一定削弱技能，关键看你是否主动去理解

### 5.3 Interactive Explanation（交互式解释）

- **场景**：线性走读后仍缺乏直觉理解（如词云的"阿基米德螺旋放置算法"）
- **方法**：让 Agent 构建动画交互页面，可视化算法执行过程
- 支持速度滑块、逐帧步进、PNG 导出
- **核心洞察**：用 AI 构建解释性工具来理解 AI 写的代码——"用 AI 理解 AI 产出"的元策略

### 5.4 GIF 优化器实战案例

端到端 Agentic Engineering 实战——将 C 语言 Gifsicle 编译为 WebAssembly 浏览器工具：

| 要素 | 做法 | 原则 |
|------|------|------|
| 文件名 | 直接指定 `gif-optimizer.html` | 简单指令引导文件组织 |
| 技术假设 | 假设 Agent 知道 Gifsicle 和 Emscripten | 对模型知识做合理假设 |
| UI 模式 | 引用已有项目的模式 | 复用已有方案（Hoard） |
| 测试 | 全程 Rodney 浏览器自动化 | 开发中持续测试 |
| 模糊指令 | "放到合适的子目录" | Agent 参考已有结构推断 |

**实战教训**：
- Agent 擅长试错型任务（如编译 C → WASM）
- 开发中持续测试比开发完再测有效得多
- 模糊方向性指令在有上下文时比精确指令更灵活
- 实时监控让你能中途纠偏

### 5.5 Prompts I Use（常用提示词）

**1. Artifacts 自定义指令（原型开发）**

> Never use React in artifacts - always plain HTML and vanilla JavaScript and CSS with minimal dependencies. CSS should be indented with two spaces and should start like this: `<style> * { box-sizing: border-box; }</style>` Inputs and textareas should be font size 16px. Font should always prefer Helvetica. JavaScript should be two space indents and start like this: `<script type="module">` Prefer Sentence case for headings.

**2. 校对提示词（写作审查）**

> You are a proofreader for posts about to be published. 1. Identify spelling mistakes and typos 2. Identify grammar mistakes 3. Watch out for repeated terms like "It was interesting that X, and it was interesting that Y" 4. Spot any logical errors or factual mistakes 5. Highlight weak arguments that could be strengthened 6. Make sure there are no empty or placeholder links

- 原则：带有观点或"我"的内容必须由人自己写，AI 只做检查不做创作

**3. Alt Text 生成提示词（无障碍访问）**

> You write alt text for any image pasted in by the user. Alt text is always presented in a fenced code block to make it easy to copy and paste out. It is always presented on a single line so it can be used easily in Markdown images. All text on the image must be exactly included. A short note describing the nature of the image itself should go first.

**共同特征**：都很短，但每条规则都是踩坑后提炼；体现"LLM 增强人的工作，不替代创意和分析决策"。

---

## 全文核心金字塔

```
            ┌─────────────┐
            │  你的判断力   │  ← 决定写什么、为什么写
            ├─────────────┤
            │  理解与验证   │  ← 走读、交互解释、手动测试
            ├─────────────┤
            │  工程纪律    │  ← TDD、测试先行、代码审查
            ├─────────────┤
            │  Agent 执行  │  ← 生成代码、编译、重构
            └─────────────┘
```

Agent 在底层执行，人在顶层决策。中间两层（工程纪律 + 理解验证）是桥梁，确保 Agent 的产出真正变成你的能力。
