# 05_agent_methods

本目录用于系统学习和跟踪 OpenAI、Anthropic 及其他主流机构公开的 Agent 构建方法。

## 本模块关注什么

重点不是“AI Agent 是什么”的泛泛科普，而是拆解公开材料中真正可复用的方法：

1. Workflow：固定路径、多步骤、可控、可评估的工作流。
2. Agent：模型动态规划、调用工具、根据环境反馈迭代的自主流程。
3. Tool use：模型如何调用外部工具、函数、MCP、代码执行、文件检索、浏览器/电脑操作等。
4. Skills：把组织流程、专业知识、脚本、模板封装成可复用能力包。
5. Orchestration：多 agent、handoff、routing、parallelization、evaluator-optimizer 等编排模式。
6. Guardrails / Human review：权限、审批、安全边界、失败恢复和人工复核。
7. Evaluation：如何评估 agent 工作流是否稳定、可靠、可上线。

## 当前核心问题

- OpenAI 的 Responses API、Agents SDK、Tools、Skills 分别解决什么问题？
- Anthropic 为什么强调“先 workflow，后 agent”，以及 workflow 和 agent 的边界在哪里？
- Skill 与 prompt、tool、MCP、知识库、插件有什么区别？
- 对法律行业来说，哪些任务适合 workflow，哪些任务才适合 agent？
- 合同审查、法律检索、案例分析、合规资讯追踪，应如何映射到这些方法？

## 建议阅读顺序

1. `2026-06-23_openai_anthropic_agent_methods_map.md`：先看总图。
2. `sources.md`：查看官方来源清单。
3. 后续新增专题：Responses API、Anthropic workflows、Skills、Claude Code / Codex、法律场景映射。

## 初步判断

Anthropic 的公开方法论更适合作为“理解 agent 架构”的入门框架，因为它直接区分了 workflow 和 agent，并给出 prompt chaining、routing、parallelization、orchestrator-workers、evaluator-optimizer 等模式。

OpenAI 的公开材料更偏“产品化接口和工程组件”：Responses API 作为 agent-like 应用的新 API primitive，Agents SDK 处理编排、状态、审批和观测，Tools / MCP / Skills / Shell / Computer Use 则提供行动能力。

对法律行业工作流来说，第一阶段不应急着做完全自主 agent，而应先做可控 workflow：拆步骤、定输入输出、加校验、留人工复核；当任务路径无法预先固定时，再逐步引入 agent。