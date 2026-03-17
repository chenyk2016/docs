# 在技能中使用脚本

> 如何在技能中运行命令和捆绑可执行脚本。

技能可以指示智能体运行 shell 命令并在 `scripts/` 目录中捆绑可重用脚本。本指南涵盖一次性命令、具有自己依赖项的自包含脚本，以及如何为智能体使用设计脚本接口。

## 一次性命令

当现有包已经满足你的需求时，你可以直接在 `SKILL.md` 指令中引用它，而不需要 `scripts/` 目录。许多生态系统提供在运行时自动解析依赖项的工具。

### uvx

[uvx](https://docs.astral.sh/uv/guides/tools/) 在隔离环境中运行 Python 包，具有积极的缓存。它随 [uv](https://docs.astral.sh/uv/) 一起提供。

```bash
uvx ruff@0.8.0 check .
uvx black@24.10.0 .
```

- 不随 Python 捆绑——需要单独安装。
- 快速。积极缓存，因此重复运行几乎是即时的。

### pipx

[pipx](https://pipx.pypa.io/) 在隔离环境中运行 Python 包。可通过操作系统包管理器获得（`apt install pipx`、`brew install pipx`）。

```bash
pipx run 'black==24.10.0' .
pipx run 'ruff==0.8.0' check .
```

- 不随 Python 捆绑——需要单独安装。
- `uvx` 的成熟替代方案。虽然 `uvx` 已成为标准推荐，但 `pipx` 仍然是可靠的选择，具有更广泛的操作系统包管理器可用性。

### npx

[npx](https://docs.npmjs.com/cli/commands/npx) 运行 npm 包，按需下载。它随 npm 一起提供（npm 随 Node.js 一起提供）。

```bash
npx eslint@9 --fix .
npx create-vite@6 my-app
```

- 随 Node.js 捆绑——不需要额外安装。
- 下载包，运行它，并缓存以供将来使用。
- 使用 `npx package@version` 固定版本以确保可重复性。

### bunx

[bunx](https://bun.sh/docs/cli/bunx) 是 Bun 的 npx 等效物。它随 [Bun](https://bun.sh/) 一起提供。

```bash
bunx eslint@9 --fix .
bunx create-vite@6 my-app
```

- 在基于 Bun 的环境中直接替代 `npx`。
- 仅当用户环境有 Bun 而不是 Node.js 时才适用。

### deno run

[deno run](https://docs.deno.com/runtime/reference/cli/run/) 直接从 URL 或说明符运行脚本。它随 [Deno](https://deno.com/) 一起提供。

```bash
deno run npm:create-vite@6 my-app
deno run --allow-read npm:eslint@9 -- --fix .
```

- 文件系统/网络访问需要权限标志（`--allow-read` 等）。
- 使用 `--` 分隔 Deno 标志和工具自己的标志。

### go run

[go run](https://pkg.go.dev/cmd/go#hdr-Compile_and_run_Go_program) 直接编译并运行 Go 包。它内置于 `go` 命令中。

```bash
go run golang.org/x/tools/cmd/goimports@v0.28.0 .
go run github.com/golangci/golangci-lint/cmd/golangci-lint@v1.62.0 run
```

- 内置于 Go——不需要额外工具。
- 固定版本或使用 `@latest` 使命令明确。

**技能中一次性命令的提示：**

- **固定版本**（例如，`npx eslint@9.0.0`）使命令随时间保持一致。
- 在 `SKILL.md` 中**说明先决条件**（例如，"需要 Node.js 18+"），而不是假设智能体环境有它们。对于运行时级要求，使用 [`compatibility` 前置数据字段](../specification.md#compatibility-字段)。
- **将复杂命令移入脚本。** 一次性命令在用几个标志调用工具时工作良好。当命令变得复杂到难以一次做对时，`scripts/` 中的测试脚本更可靠。

## 从 `SKILL.md` 引用脚本

使用从技能目录根目录开始的**相对路径**来引用捆绑文件。智能体自动解析这些路径——不需要绝对路径。

在 `SKILL.md` 中列出可用脚本，让智能体知道它们存在：

```markdown SKILL.md
## 可用脚本

- **`scripts/validate.sh`** —— 验证配置文件
- **`scripts/process.py`** —— 处理输入数据
```

然后指示智能体运行它们：

```markdown SKILL.md
## 工作流

1. 运行验证脚本：
   ```bash
   bash scripts/validate.sh "$INPUT_FILE"
   ```

2. 处理结果：
   ```bash
   python3 scripts/process.py --input results.json
   ```
```

相同的相对路径约定适用于 `references/*.md` 等支持文件——代码块中的脚本执行路径相对于**技能目录根目录**，因为智能体从那里运行命令。

## 自包含脚本

当你需要可重用逻辑时，在 `scripts/` 中捆绑一个脚本，该脚本内联声明自己的依赖项。智能体可以用单个命令运行脚本——不需要单独的清单文件或安装步骤。

几种语言支持内联依赖声明：

### Python

[PEP 723](https://peps.python.org/pep-0723/) 定义了内联脚本元数据的标准格式。在 `# ///` 标记内的 TOML 块中声明依赖项：

```python scripts/extract.py
# /// script
# dependencies = [
#   "beautifulsoup4",
# ]
# ///

from bs4 import BeautifulSoup

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
print(BeautifulSoup(html, "html.parser").select_one("p.info").get_text())
```

使用 [uv](https://docs.astral.sh/uv/) 运行（推荐）：

```bash
uv run scripts/extract.py
```

`uv run` 创建隔离环境，安装声明的依赖项，并运行脚本。[pipx](https://pipx.pypa.io/)（`pipx run scripts/extract.py`）也支持 PEP 723。

- 使用 [PEP 508](https://peps.python.org/pep-0508/) 说明符固定版本：`"beautifulsoup4>=4.12,<5"`。
- 使用 `requires-python` 约束 Python 版本。
- 使用 `uv lock --script` 创建锁定文件以确保完全可重复性。

### Deno

Deno 的 `npm:` 和 `jsr:` 导入说明符使每个脚本默认自包含：

```typescript scripts/extract.ts
#!/usr/bin/env -S deno run

import * as cheerio from "npm:cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

```bash
deno run scripts/extract.ts
```

- 对 npm 包使用 `npm:`，对 Deno 原生包使用 `jsr:`。
- 版本说明符遵循 semver：`@1.0.0`（精确）、`@^1.0.0`（兼容）。
- 依赖项全局缓存。使用 `--reload` 强制重新获取。
- 具有原生插件（node-gyp）的包可能无法工作——提供预构建二进制文件的包工作最佳。

### Bun

当没有找到 `node_modules` 目录时，Bun 在运行时自动安装缺失的包。直接在导入路径中固定版本：

```typescript scripts/extract.ts
#!/usr/bin/env bun

import * as cheerio from "cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

```bash
bun run scripts/extract.ts
```

- 不需要 `package.json` 或 `node_modules`。TypeScript 原生工作。
- 包全局缓存。第一次运行下载；后续运行几乎是即时的。
- 如果目录树中的任何位置存在 `node_modules` 目录，自动安装被禁用，Bun 回退到标准 Node.js 解析。

### Ruby

Bundler 自 Ruby 2.6 起随 Ruby 一起提供。使用 `bundler/inline` 直接在脚本中声明 gem：

```ruby scripts/extract.rb
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'nokogiri'
end

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
doc = Nokogiri::HTML(html)
puts doc.at_css('p.info').text
```

```bash
ruby scripts/extract.rb
```

- 显式固定版本（`gem 'nokogiri', '~> 1.16'`）——没有锁定文件。
- 工作目录中现有的 `Gemfile` 或 `BUNDLE_GEMFILE` 环境变量可能干扰。

## 为智能体使用设计脚本

当智能体运行你的脚本时，它读取 stdout 和 stderr 来决定下一步做什么。一些设计选择使脚本对智能体使用更容易。

### 避免交互式提示

这是智能体执行环境的硬性要求。智能体在非交互式 shell 中操作——它们无法响应 TTY 提示、密码对话框或确认菜单。阻塞在交互式输入上的脚本将无限期挂起。

通过命令行标志、环境变量或 stdin 接受所有输入：

```
# 不好：挂起等待输入
$ python scripts/deploy.py
目标环境：_

# 好：带有指导的清晰错误
$ python scripts/deploy.py
错误：需要 --env。选项：development、staging、production。
用法：python scripts/deploy.py --env staging --tag v1.2.3
```

### 用 `--help` 记录用法

`--help` 输出是智能体学习脚本接口的主要方式。包括简要描述、可用标志和用法示例：

```
用法：scripts/process.py [选项] INPUT_FILE

处理输入数据并生成汇总报告。

选项：
  --format FORMAT    输出格式：json、csv、table（默认：json）
  --output FILE      将输出写入 FILE 而不是 stdout
  --verbose          将进度打印到 stderr

示例：
  scripts/process.py data.csv
  scripts/process.py --format csv --output report.csv data.csv
```

保持简洁——输出进入智能体的上下文窗口，与它正在处理的所有其他内容一起。

### 编写有用的错误消息

当智能体收到错误时，消息直接塑造它的下一次尝试。不透明的"错误：无效输入"浪费一个回合。相反，说明出了什么问题、期望什么以及尝试什么：

```
错误：--format 必须是以下之一：json、csv、table。
       收到："xml"
```

### 使用结构化输出

优先选择结构化格式——JSON、CSV、TSV——而不是自由格式文本。结构化格式可以被智能体和标准工具（`jq`、`cut`、`awk`）消费，使你的脚本在管道中可组合。

```
# 空白对齐——难以程序化解析
NAME          STATUS    CREATED
my-service    running   2025-01-15

# 分隔——明确的字段边界
{"name": "my-service", "status": "running", "created": "2025-01-15"}
```

**将数据与诊断分开：** 将结构化数据发送到 stdout，将进度消息、警告和其他诊断发送到 stderr。这让智能体在需要时捕获干净、可解析的输出，同时仍然可以访问诊断信息。

### 进一步考虑

- **幂等性。** 智能体可能重试命令。"如果不存在则创建"比"创建并在重复时失败"更安全。
- **输入约束。** 拒绝模糊输入并带有清晰错误，而不是猜测。尽可能使用枚举和闭集。
- **Dry-run 支持。** 对于破坏性或状态操作，`--dry-run` 标志让智能体预览将要发生的情况。
- **有意义的退出代码。** 为不同的失败类型使用不同的退出代码（未找到、无效参数、身份验证失败），并在 `--help` 输出中记录它们，让智能体知道每个代码的含义。
- **安全默认值。** 考虑破坏性操作是否应该需要显式确认标志（`--confirm`、`--force`）或适合风险级别的其他保护措施。
- **可预测的输出大小。** 许多智能体工具自动截断超过阈值（例如 10-30K 字符）的工具输出，可能丢失关键信息。如果你的脚本可能产生大量输出，默认为摘要或合理限制，并支持 `--offset` 等标志让智能体在需要时请求更多信息。或者，如果输出很大且不适合分页，要求智能体传递指定输出文件或 `-` 以明确选择 stdout 的 `--output` 标志。
