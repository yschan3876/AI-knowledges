# Agentic Engineering Patterns — 团队分享

> 基于 Simon Willison（Django 联合创始人）的实战指南
> 原文：[Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/)

---

## 一句话总结

**写代码变便宜了，但交付好代码依然昂贵。** Agentic Engineering 就是在 AI 编码 Agent 辅助下，依然保持工程纪律、交付生产级代码的方法论。

---

## 为什么你应该关注这个话题

我们每天都在用 Claude Code、Cursor 等 AI 编码工具。但很少有人系统思考过：**怎样才算用对了？**

Simon Willison 给出了一个关键区分：

| | Vibe Coding | Agentic Engineering |
|---|---|---|
| 定义 | 让 AI 写代码，不仔细审查，快速出原型 | 让 AI 写代码，但你负责把它带到生产级别 |
| 适用场景 | 一次性脚本、快速验证想法 | 任何要上线、要维护的代码 |
| 核心区别 | **你不对产出负责** | **你对每一行代码负责** |

> 大多数人以为自己在做 Agentic Engineering，实际上在 Vibe Coding。

---

## 5 个核心 Insight

### Insight 1: 你的价值不再是"写代码"，而是"决定写什么"

Agent 能写代码，但不能替你做判断：
- 这个问题该用缓存还是预计算？
- 这个 API 该同步还是异步？
- 这个 feature 值不值得做？

**每个问题有多种解法和不同权衡，这部分判断力 Agent 替代不了。**

行动建议：把精力从"怎么实现"转移到"实现什么"和"为什么这样实现"上。

### Insight 2: 囤积知识，让它产生复利

> "每个技巧只需搞懂一次，记下来，Agent 可以无限复用。"

这是整篇文章最有实操价值的建议：

1. **记录你会的东西** — 博客、TIL、GitHub 仓库、甚至一个单文件工具，形式不重要
2. **告诉 Agent 组合它们** — "结合方案 A 和方案 B 构建新东西"
3. **享受复利** — 你的技能库越大 × Agent 的执行力 = 指数级产出

案例：Simon 告诉 Agent "用 Tesseract.js 做 OCR + PDF.js 解析 PDF"，Agent 就能组合出一个 PDF OCR 工具。他只需要知道这两个库存在。

### Insight 3: 三层测试防线，一个都不能少

这是全文最体系化的部分：

```
第一层：Red/Green TDD
  ↓ 防止代码不工作 + 防止多写不需要的代码
第二层：First Run the Tests
  ↓ 让 Agent 意识到测试存在，建立测试意识
第三层：手动测试
  ↓ 防止"测试通过但功能不对" + 防止 Agent 伪造测试结果
```

两个魔法提示词：
- **`Use red/green TDD`** — 6 个字，编码完整的 TDD 工程纪律
- **`First run the tests`** — 每次打开已有项目时用，三重效果

关键认知：**通过测试 ≠ 按预期工作。** 发布前必须用自己的眼睛看到功能在工作。

### Insight 4: 头号反模式 — 提交没 review 过的 PR

> 这是 Agentic Engineering 时代的新职业素养问题。

一个好的 Agentic PR 应该：
- 代码能工作，且你有信心（不是"Agent 说能工作"）
- 变更足够小，容易审查
- PR 描述里附上**工作证据**：测试截图、视频、实现选择说明

**证明你审查过，而不是让 reviewer 替你审查 Agent 的产出。**

### Insight 5: 警惕认知债务

Agent 帮你写了代码，但你不理解它是怎么工作的 — 这就是**认知债务（Cognitive Debt）**。

- 简单工具代码：可以不深究
- 复杂业务逻辑：**必须理解**，否则无法自信地做后续变更

两种"还债"方法：
1. **线性走读** — 让 Agent 生成逐文件的结构化走读文档
2. **交互式解释** — 让 Agent 构建可视化页面，动画展示算法执行过程

核心洞察：**用 AI 构建解释性工具来理解 AI 写的代码** — 这是一种元策略。

---

## 全文核心模型

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

**Agent 在底层执行，人在顶层决策。** 中间两层是桥梁 — 没有它们，Agent 的产出无法真正变成你的能力，你只是在 Vibe Coding。

---

## 马上可以做的 3 件事

1. **今天起，每次用 Agent 打开项目先说 `First run the tests`** — 零成本，立刻建立 Agent 的测试意识

2. **开始记录你的技巧库** — 哪怕只是一个 Markdown 文件，记下"我知道怎么做 X"。下次告诉 Agent 组合它们，你会惊讶于产出效率

3. **提交 PR 前问自己："如果有人问我这段代码为什么这样写，我能回答吗？"** — 如果不能，你欠了认知债务，先还债再提交

---

## 延伸阅读

- [原文全文](https://simonwillison.net/guides/agentic-engineering-patterns/) — Simon Willison 的完整指南，包含更多案例和细节
- [完整学习笔记](./agentic-engineering-patterns.md) — 15 个章节的逐章拆解笔记
