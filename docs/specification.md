# 规范

Agent Skills 的完整格式规范。

## 目录结构

一个 skill 是一个目录，至少包含一个 `SKILL.md` 文件：

```
skill-name/
├── SKILL.md          # 必需：元数据 + 指令
├── scripts/          # 可选：可执行代码
├── references/       # 可选：文档
├── assets/           # 可选：模板、资源
└── ...               # 任何其他文件或目录
```

## `SKILL.md` 格式

`SKILL.md` 文件必须包含 YAML 前置数据，后跟 Markdown 内容。

### 前置数据

| 字段 | 必需 | 约束 |
|------|------|------|
| `name` | 是 | 最多 64 个字符。仅小写字母、数字和连字符。不能以连字符开头或结尾。 |
| `description` | 是 | 最多 1024 个字符。非空。描述 skill 的作用以及何时使用它。 |
| `license` | 否 | 许可证名称或对捆绑许可证文件的引用。 |
| `compatibility` | 否 | 最多 500 个字符。指示环境要求（目标产品、系统包、网络访问等）。 |
| `metadata` | 否 | 用于附加元数据的任意键值映射。 |
| `allowed-tools` | 否 | skill 可使用的预批准工具的空格分隔列表。（实验性） |

**最小示例：**

```markdown SKILL.md
---
name: skill-name
description: 描述此 skill 的作用以及何时使用它。
---
```

**包含可选字段的示例：**

```markdown SKILL.md
---
name: pdf-processing
description: 提取 PDF 文本、填写表单、合并文件。处理 PDF 时使用。
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

#### `name` 字段

必需的 `name` 字段：
- 必须为 1-64 个字符
- 只能包含 Unicode 小写字母数字字符（`a-z`）和连字符（`-`）
- 不能以连字符（`-`）开头或结尾
- 不能包含连续连字符（`--`）
- 必须与父目录名称匹配

**有效示例：**
```yaml
name: pdf-processing
```
```yaml
name: data-analysis
```
```yaml
name: code-review
```

**无效示例：**
```yaml
name: PDF-Processing  # 不允许大写
```
```yaml
name: -pdf  # 不能以连字符开头
```
```yaml
name: pdf--processing  # 不允许连续连字符
```

#### `description` 字段

必需的 `description` 字段：
- 必须为 1-1024 个字符
- 应描述 skill 的作用以及何时使用它
- 应包含帮助智能体识别相关任务的特定关键词

**好示例：**
```yaml
description: 从 PDF 文件中提取文本和表格，填写 PDF 表单，并合并多个 PDF。处理 PDF 文档或当用户提到 PDF、表单或文档提取时使用。
```

**差示例：**
```yaml
description: 帮助处理 PDF。
```

#### `license` 字段

可选的 `license` 字段：
- 指定应用于 skill 的许可证
- 建议保持简短（许可证名称或捆绑许可证文件的名称）

**示例：**
```yaml
license: 专有。LICENSE.txt 包含完整条款
```

#### `compatibility` 字段

可选的 `compatibility` 字段：
- 如果提供，必须为 1-500 个字符
- 仅当 skill 有特定环境要求时才应包含
- 可指示目标产品、必需的系统包、网络访问需求等

**示例：**
```yaml
compatibility: 专为 Claude Code（或类似产品）设计
```
```yaml
compatibility: 需要 git、docker、jq 和互联网访问
```

大多数 skills 不需要 `compatibility` 字段。

#### `metadata` 字段

可选的 `metadata` 字段：
- 从字符串键到字符串值的映射
- 客户端可以使用此字段存储 Agent Skills 规范未定义的附加属性
- 建议使键名足够独特以避免意外冲突

**示例：**
```yaml
metadata:
  author: example-org
  version: "1.0"
```

#### `allowed-tools` 字段

可选的 `allowed-tools` 字段：
- 预批准可运行的工具的空格分隔列表
- 实验性。不同智能体实现对此字段的支持可能有所不同

**示例：**
```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read
```

### 正文内容

前置数据之后的 Markdown 正文包含 skill 指令。没有格式限制。编写任何有助于智能体有效执行任务的内容。

推荐的部分：
- 分步说明
- 输入和输出示例
- 常见边界情况

注意，一旦智能体决定激活 skill，就会加载整个文件。考虑将较长的 `SKILL.md` 内容拆分到引用的文件中。

## 可选目录

### `scripts/`

包含智能体可以运行的可执行代码。脚本应该：
- 自包含或清楚地记录依赖项
- 包含有用的错误消息
- 优雅地处理边界情况

支持的语言取决于智能体实现。常见选项包括 Python、Bash 和 JavaScript。

### `references/`

包含智能体在需要时可以阅读的附加文档：
- `REFERENCE.md` - 详细技术参考
- `FORMS.md` - 表单模板或结构化数据格式
- 领域特定文件（`finance.md`、`legal.md` 等）

保持单个[引用文件](#文件引用)聚焦。智能体按需加载这些文件，因此较小的文件意味着更少的上下文使用。

### `assets/`

包含静态资源：
- 模板（文档模板、配置模板）
- 图片（图表、示例）
- 数据文件（查找表、模式）

## 渐进式披露

Skills 应该为高效使用上下文而构建：

1. **元数据**（约 100 个 token）：`name` 和 `description` 字段在启动时为所有 skills 加载
2. **指令**（建议 < 5000 个 token）：当 skill 激活时加载完整的 `SKILL.md` 正文
3. **资源**（按需）：文件（例如 `scripts/`、`references/` 或 `assets/` 中的文件）仅在需要时加载

保持主 `SKILL.md` 在 500 行以内。将详细的参考资料移到单独的文件中。

## 文件引用

在 skill 中引用其他文件时，使用相对于 skill 根目录的路径：

```markdown SKILL.md
详见[参考指南](references/REFERENCE.md)。

运行提取脚本：
scripts/extract.py
```

保持文件引用从 `SKILL.md` 开始只有一层深度。避免深度嵌套的引用链。

## 验证

使用 [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) 参考库验证你的 skills：

```bash
skills-ref validate ./my-skill
```

这将检查你的 `SKILL.md` 前置数据是否有效并遵循所有命名约定。
