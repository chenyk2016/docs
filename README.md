# Agent Skills 中文文档

[Agent Skills](https://agentskills.io) 是一个简单、开放的格式，用于赋予 AI 智能体新的能力和专业知识。

这是一个 Mintlify 文档项目，包含 Agent Skills 的完整中文翻译文档。

## 开始使用

### 本地预览

```bash
# 安装 Mintlify CLI
npm i -g mintlify

# 进入项目目录
cd agentskills-zh

# 启动本地预览
mint dev
```

预览将在 `http://localhost:3000` 打开。

### 部署

此文档托管在 Mintlify 上。推送到 main 分支会自动触发部署。

## 文档结构

```
agentskills-zh/
├── docs.json              # Mintlify 配置文件
├── favicon.svg           # 网站图标
├── logo/                 # Logo 文件
│   ├── light.svg
│   └── dark.svg
├── docs/                 # 文档内容
│   ├── index.mdx         # 首页/概述
│   ├── what-are-skills.mdx
│   ├── specification.mdx
│   ├── skill-creation/   # 技能创建指南
│   │   ├── best-practices.mdx
│   │   ├── evaluating-skills.mdx
│   │   ├── optimizing-descriptions.mdx
│   │   └── using-scripts.mdx
│   └── client-implementation/  # 客户端实现
│       └── adding-skills-support.mdx
└── README.md
```

## 翻译内容

- **概述** - Agent Skills 简介和核心概念
- **什么是技能？** - 技能的工作原理和基本结构
- **规范** - SKILL.md 文件的完整格式规范
- **最佳实践** - 编写高质量技能的指南
- **评估技能** - 如何测试和迭代技能
- **优化描述** - 改进技能描述以提高触发准确性
- **使用脚本** - 在技能中运行命令和脚本
- **添加技能支持** - 为智能体产品添加技能支持的实现指南

## 许可证

本文档采用 [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/) 许可证。

原始 Agent Skills 项目由 [Anthropic](https://anthropic.com) 维护。
