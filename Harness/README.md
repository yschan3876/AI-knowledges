# Harness 设计

> AI Agent 脚手架系统（Harness）的设计与迭代方法论

## 内容导航

| 文档 | 主题 | 来源 |
|------|------|------|
| [Harness Design for Long-Running Apps](./harness-design-long-running-apps.md) | 多 Agent 架构、Context 管理、Sprint Contract、Load-Bearing 分析 | [Anthropic Engineering Blog](https://www.anthropic.com/engineering/harness-design-long-running-apps) |

---

## 分类说明

**Harness** 在 AI Agent 语境下特指围绕模型的**脚手架系统**：分解工作、管理上下文、提供结构化反馈的编排层。本分类关注：

- **多 Agent 架构设计**：Generator / Evaluator / Planner 等角色分工
- **Context 管理**：Compaction、Context Reset、结构化 Handoff
- **评分机制**：把主观判断转化为可评分标准
- **迭代方法论**：Load-Bearing 分析、随模型能力演进的简化策略

Harness 设计的核心哲学：**每个组件都是对模型弱点的补偿机制，随模型升级会重新评估其必要性**。
