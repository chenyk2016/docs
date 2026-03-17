# 概述

> 一个简单、开放的格式，用于赋予 AI 智能体新的能力和专业知识。

Agent Skills 是包含指令、脚本和资源的文件夹，智能体可以发现并使用它们来更准确、更高效地完成任务。

## 为什么需要 Agent Skills？

智能体的能力越来越强，但往往缺乏可靠地完成实际工作所需的上下文。技能通过让智能体访问程序性知识和公司、团队、用户特定的上下文来解决这个问题，这些上下文可以根据需要加载。拥有技能集的智能体可以根据正在处理的任务扩展其能力。

**对于技能作者**：构建一次能力，部署到多个智能体产品。

**对于兼容的智能体**：支持技能让终端用户能够为智能体提供开箱即用的新能力。

**对于团队和企业**：将组织知识捕获到可移植、版本控制的包中。

## Agent Skills 能实现什么？

- **领域专业知识**：将专业知识打包成可重用的指令，从法律审查流程到数据分析管道。
- **新能力**：赋予智能体新能力（例如创建演示文稿、构建 MCP 服务器、分析数据集）。
- **可重复的工作流**：将多步骤任务转化为一致且可审计的工作流。
- **互操作性**：在不同的技能兼容智能体产品之间重用相同的技能。

## 采用情况

Agent Skills 得到领先的 AI 开发工具支持，包括：
- Claude Code / Claude
- OpenAI Codex
- GitHub Copilot / VS Code
- Cursor
- Gemini CLI
- OpenHands
- Roo Code
- 以及更多...

## 开放开发

Agent Skills 格式最初由 [Anthropic](https://www.anthropic.com/) 开发，作为开放标准发布，并已被越来越多的智能体产品采用。该标准欢迎更广泛的生态系统贡献。

[在 GitHub 上查看](https://github.com/agentskills/agentskills)

## 开始使用

- [什么是技能？](what-are-skills.md) - 了解技能的工作原理及其重要性
- [规范](specification.md) - SKILL.md 文件的完整格式规范
- [添加技能支持](client-implementation/adding-skills-support.md) - 为你的智能体或工具添加技能支持
- [示例技能](https://github.com/anthropics/skills) - 在 GitHub 上浏览示例技能
- [参考库](https://github.com/agentskills/agentskills/tree/main/skills-ref) - 验证技能并生成提示 XML
