# Claude Skill YAML Frontmatter 字段速查

> 写 SKILL.md 时对照此表。文件位置: 与 Claude-Skill_zh-CN.xml 同目录。
> 每字段标注了来源可信度：
>   🔵 官方确认 — code.claude.com/docs/en/skills
>   🟢 开放标准 — agentskills.io，Claude Code 兼容
>   ⚠️ 社区使用 — 官方未确认
> 实测列：✅ 行为已验证 / ⚠️ 部分有效 / ❓ 环境受限未测全

---

## 一、字段总览

### 实测结论汇总（共 21 个字段）

| 结果 | 数量 | 字段 |
|------|------|------|
| ✅ 行为已验证 | 17 | name、description、argument-hint、arguments、allowed-tools、when_to_use、context、agent、model、effort、shell、disable-model-invocation、license、compatibility、metadata、dependencies、paths |
| ⚠️ 部分有效 | 2 | user-invocable（菜单隐藏生效，Skill工具仍可调用）、hooks（注册成功但拦截效果未完全确认） |
| ❓ 环境受限未测全 | 1 | args（arguments别名） |
| ❌ 无效 | 1 | arguments[].required:true（不做强制校验，不传参也能执行） |

---

## 二、字段详解

### 2.1 核心标识

| 字段           | 类型   | 必备 | 来源    | 实测 | 说明                                                                 |
|---------------|------  |-----|--------|------|---------------------------------------------------------------|
| `name`        | string | 推荐 | 🔵 官方 | ✅ | skill 名称，kebab-case，最长64字符。也是 `/name` 命令名。省略时取目录名 |
| `description` | string | 推荐 | 🔵 官方 | ✅ | 功能描述，最长1024字符。Claude 用此判断触发时机 |

写法：
```yaml
name: my-skill
description: 简短说清做什么和什么时候用

# 多行描述用 | 符号（来源: GitHub issue#11322）
description: |
  第一行概述。
  第二行补充。
```

### 2.2 参数提示

这三个字段控制 skill 如何接收用户输入的参数。

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `argument-hint` | string | 可选 | 🔵 官方 | ✅ | 用户在终端敲 `/skill ` 时，提示他该输什么。如 `[issue-number]` |
| `arguments` | list/string | 可选 | 🔵 官方 | ✅ | 命名位置参数。空格分隔字符串或 YAML list。正文用 `$name` 引用。`required: true` 不做强制校验 |
| `args` | list | 可选 | ⚠️ 社区 | ❓ | arguments 的别名，格式相同 |

#### argument-hint

仅做视觉提示，不校验用户是否输入。

```yaml
argument-hint: "[issue-number]"
```
用户输入 `/fix-issue ` 后终端显示：
```
fix-issue [issue-number]
```

#### arguments

定义参数结构，skill 正文中用 `$name` 引用。

```yaml
arguments:
  - name: component
    description: 要迁移的组件名
    required: true
  - name: from
    description: 源框架
    required: true
  - name: to
    description: 目标框架
    required: false
```

#### args

arguments 的别名，选一个用，不要同时写两个：

```yaml
args:
  - name: input
    description: 输入文件
    required: true
```

#### 参数引用方式（来源: 官方文档）

skill 正文中可以用以下方式引用参数：

```
$ARGUMENTS          → "SearchBar React Vue"（完整字符串，空格分隔）
$ARGUMENTS[0] / $0  → "SearchBar"（第1个参数，从0开始）
$ARGUMENTS[1] / $1  → "React"（第2个参数）
$component          → "SearchBar"（命名参数，arguments 中定义的 name）
```

#### 实测注意

- `required: true` **不做强制校验**，不传参也能执行
- `args` 与 `arguments` 等价，逻辑上等效

### 2.3 调用控制

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `user-invocable` | boolean | 可选 | 🔵 官方 | ⚠️ | `false` = 从 `/` 菜单隐藏，仅 Claude 自动触发。缺省 `true`。实测：菜单隐藏生效，但 Skill 工具仍可调用 |
| `disable-model-invocation` | boolean | 可选 | 🔵 官方 | ✅ | `true` = 禁止 Claude 自动调用，仅手动 `/name` 触发。缺省 `false` |

写法：
```yaml
user-invocable: true           # true = 显示在 / 菜单（默认）
user-invocable: false          # false = 隐藏，仅Claude自动触发

disable-model-invocation: true   # true = 禁止自动调用，仅手动 /name
disable-model-invocation: false  # false = 允许自动调用（默认）
```

三种调用模式（来源: 官方文档）：

| disable-model-invocation | user-invocable | 用户能调 | Claude 能调 | 描述加载 |
|---|---|---|---|---|
| false(默认) | true(默认) | ✅ /name | ✅ 自动 | ✅ 始终 |
| true | — | ✅ /name | ❌ | ❌ 不加载 |
| — | false | ❌ 隐藏 | ✅ 自动 | ✅ 始终 |

### 2.4 执行环境

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `context` | string | 可选 | 🔵 官方 | ✅ | `fork` = 在隔离子代理中运行，不污染主对话上下文 |
| `agent` | string | 可选 | 🔵 官方 | ✅ | 子代理类型（仅 `context: fork` 时生效）。`general-purpose`(默认) / `Explore`(只读) / `Plan`(规划) |
| `model` | string | 可选 | 🔵 官方 | ❌ | 模型覆盖。实测：你后端接的是 DeepSeek，设 haiku/sonnet/opus 全被忽略，始终用 deepseek-v4-flash。仅对 Claude 原生 API 生效 |
| `effort` | string | 可选 | 🔵 官方 | ✅ | 推理力度：`low` / `medium` / `high` / `xhigh` / `max`。实测有效：low 简洁概括，max 深度结构化分析。即使是 DeepSeek 后端也生效（effort 是 Claude Code 客户端行为，不依赖 API） |
写法：
```yaml
context: fork             # fork = 隔离子代理。不写=默认内联执行
agent: Explore            # 仅 context:fork 时生效。Explore/Plan/general-purpose
model: haiku              # haiku(轻量) / sonnet(日常) / opus(深度) / 完整模型名
effort: medium            # low/medium/high/xhigh/max
```

### 2.5 工具控制

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `allowed-tools` | list/string | 可选 | 🔵 官方 | ✅ (有 bug) | 免审批工具白名单。支持精确工具名和作用域限定。格式：空格/逗号分隔字符串，或 YAML list。⚠️ 社区反馈 [#18837](https://github.com/anthropics/claude-code/issues/18837)：有时不被执行，Claude 仍可用列表外的工具 |
| `~~disallowed-tools~~` | — | — | ❌ 不存在的字段 | ❌ | SKILL.md 中 **无此字段**。常用于 settings.json 的 `disallowedTools`（注意驼峰命名）被误传为 SKILL.md 字段。官方 help center + Skills v2.0 完整字段表均未收录此字段 |

#### 可用工具名（来源: 官方 CLI）

```
Read  Write  Edit  Grep  Glob  Bash  Task  TaskCreate
TaskUpdate  TaskList  TaskGet  AskUserQuestion  WebSearch  WebFetch
```

#### allowed-tools 写法一：裸工具名（不限定范围）

```yaml
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash       # 等于开放所有 shell 命令，谨慎使用
```

#### allowed-tools 写法二：作用域限定（推荐，来源: 官方 settings.json 格式）

```yaml
allowed-tools:
  - Bash(git:*)        # 冒号语法。只允许 git 命令
  - Bash(npm:install)  # 只允许 npm install
  - Bash(python:*)     # 只允许 python 命令
```

也有社区使用空格语法 `Bash(git *)`，两者等效。

#### allowed-tools 写法三：MCP 工具（来源: 官方文档）

```yaml
allowed-tools:
  - mcp__github__*              # GitHub MCP 全部工具
  - mcp__linear__create_issue   # 单个 MCP 工具
```

#### ⚠️ disallowed-tools 误区澄清

`disallowed-tools` **不是 SKILL.md 的官方字段**。经查：

- Anthropic 官方 [Help Center](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills) 仅收录 `name`、`description`、`dependencies`
- Skills v2.0 完整字段表（19 个字段）**均未收录** `disallowed-tools`
- anthropic/claude-code GitHub 仓库中**无对应 issue** 或实现代码

此字段大概率是 `settings.json` 中的 `disallowedTools`（驼峰命名）被误传为 SKILL.md 字段。

##### 实测验证

三种格式（YAML 列表 / 逗号分隔字符串 / 空格分隔字符串）全部**无效**，工具调用完全不受影响。

##### 实际用法：settings.json 中的 disallowedTools（有效）

```json
{
  "disallowedTools": [
    "Write",
    "Bash(rm:*)",
    "WebSearch"
  ]
}
```

- 全局或项目级生效（放入 `.claude/settings.json`）
- 也可用 CLI 参数：`claude --disallowedTools "WebSearch" "Write"`
- 这是当前推荐的做法

#### 防范提示（来源: 官方工具权限文档）

- `Bash`（无限定）= 开放所有 shell 命令
- 管道命令的每段都会被独立检查：`git status && rm -rf /` 需要 Bash(git:*) + Bash(rm:*) 都通过
- 命令替换 `$(...)` 和子进程不参与模式匹配，始终弹权限
- 拒绝规则（denied-tools）优先于允许规则

### 2.6 生命周期钩子

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `hooks` | object | 可选 | 🔵 官方 | ⚠️ | 钩子定义。含 `PreToolUse` / `PostToolUse` / `Stop` / `SessionStart` 等。实测：prompt 类型钩子静默通过，command 类型受沙箱限制 |

写法（来源: 官方钩子文档）：
```yaml
hooks:
  PreToolUse:
    - matcher: "Bash|Write|Edit"     # 匹配工具名，支持正则
      type: prompt                     # prompt(推荐) / command / http
      prompt: "检查操作是否安全..."
      # 或 command: "./scripts/validate.sh"
      once: true                       # 每个会话只执行一次
      timeout: 30                      # 超时秒数
  PostToolUse:
    - matcher: "Edit"
      type: command
      command: "echo 'done'"
```

支持的事件：
- `PreToolUse` — 工具执行前，可拦截。exit 2 = 阻止
- `PostToolUse` — 工具执行后，用于反馈/记录
- `Stop` — 任务结束时，检查完成度
- `SessionStart` — 会话开始时，加载上下文
- `UserPromptSubmit` — 用户输入时，注入上下文
- `SubagentStop` — 子代理完成时

### 2.7 路径与 Shell

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `paths` | list/string | 可选 | 🔵 官方 | ✅ | glob 模式。实测：**无匹配文件时 skill 完全不加载**（自动触发 + `/` 命令均不可用）。有匹配文件时 skill 正常可用。相当于 skill 级 gate 条件 |
| `shell` | string | 可选 | 🔵 官方 | ✅ | `!command` 的 shell：`bash`（默认）或 `powershell` |

写法（来源: 官方文档 + 实测）：

```yaml
# paths：YAML list 或单字符串
paths:
  - "src/**/*.ts"
  - "test/**/*.ts"

# 或单字符串
paths: "src/**/*.ts"
```

#### paths 使用格式

##### 基本格式

| 格式 | 示例 | 说明 |
|------|------|------|
| 单字符串 | `paths: "*.py"` | 匹配当前目录下所有 .py 文件 |
| YAML 列表 | `paths: ["src/**/*.ts", "test/**/*.ts"]` | 匹配多个模式，**或**关系 |
| 多行列表 | `paths:` 换行 `  - "*.py"` | 同列表格式，推荐 |

##### 常见 glob 模式

| 模式 | 匹配范围 |
|------|---------|
| `*.md` | 当前目录的 .md 文件 |
| `**/*.py` | 任意子目录的 .py 文件 |
| `src/**/*.ts` | `src/` 目录下所有 .ts 文件 |
| `*.{py,js,ts}` | 三种扩展名 |
| `docs/*` | `docs/` 目录下任意文件（不递归） |
| `project-a/**` | `project-a/` 下所有文件 |

##### 典型使用场景

```yaml
# 只在前端项目中生效
paths: "src/**/*.{ts,tsx,js,jsx}"

# 只在包含 Python 文件的目录中生效
paths:
  - "**/*.py"
  - "requirements.txt"
  - "setup.py"

# 只在特定项目目录生效
paths: "my-project/**"

# 只在包含文档的目录生效
paths:
  - "**/*.md"
  - "**/docs/*"
```

##### 完整示例

```yaml
---
name: py-code-review
description: 自动审查 Python 代码质量，当用户提到代码审查时触发
paths:
  - "**/*.py"
---
## 使用说明

审查当前项目中的所有 Python 文件...
```

此 skill 只在包含 `.py` 文件的目录中加载，纯 JS 项目下完全不可见。

#### paths 实测结论

| 测试场景 | 匹配文件 | 自动触发 | 手工 `/` 调用 | 结论 |
|---------|---------|---------|-------------|------|
| 目录含 `*.py` 文件 | ✅ `**/*.py` 匹配 | ✅ 成功（标记可见）| ✅ 成功 | paths 匹配时 skill 正常加载 |
| 目录不含 `*.py` 文件 | ❌ 不匹配 | ❌ 未触发（通用回复）| ❌ 不识别技能 | paths 不匹配时 skill **完全不可见** |

#### 工作原理

- `paths` 是 **skill 级的 gate 条件** — 不只是限制自动触发，而是控制 skill 是否被加载到会话中
- 当当前工作目录或项目文件匹配 `paths` 的 glob 模式时，skill 可见
- 当不匹配时，skill 对 Claude 完全不可见（既不会自动触发，也无法用 `/name` 调用）
- 用于限定技能只应在特定项目中可用（如只在前端项目加载、只在 Python 项目加载）

#### 注意事项

1. **glob 匹配的是工作目录和项目文件**，不是 skill 自己目录下的文件
2. 支持 `**` 通配符递归匹配：`"src/**/*.ts"` 匹配所有子目录中的 .ts 文件
3. 列表格式的多个模式是 **或** 关系，匹配任一即可
4. `paths` 不设则 skill 在所有项目下均可用（默认行为）

#### shell 使用格式

```yaml
shell: bash          # 默认，Unix风格命令
shell: powershell    # PowerShell 命令。需要环境变量 CLAUDE_CODE_USE_POWERSHELL_TOOL=1
```

> `shell` 只影响 `!command` 语句的解析器，不影响 Bash 工具的执行环境。

### 2.8 依赖声明

| 字段 | 类型 | 实测 | 说明 |
|------|------|------|------|
| `dependencies` | list/string | ✅ AI 驱动安装 | YAML 列表或逗号分隔字符串。Claude 读到此字段后，在执行需要这些包的任务时会主动 pip install。**AI 按需安装，非 CLI 自动执行 pip** |

#### 写法一：YAML 列表（推荐）

```yaml
dependencies:
  - python>=3.10
  - pandas==2.2.2
  - matplotlib>=3.8
  - requests
```

#### 写法二：逗号分隔字符串

```yaml
dependencies: python>=3.10, pandas==2.2.2, requests
```

#### 版本约束写法

| 写法 | 含义 |
|------|------|
| `tqdm` | 任意版本 |
| `tqdm==4.67.3` | 精确版本 |
| `tqdm>=4.60` | 最低版本 |
| `tqdm>=4.60,<5.0` | 版本范围 |
| `python>=3.10` | Python 版本约束（非 pip 包） |

#### 工作机制（实测结论）

| 步骤 | 说明 |
|------|------|
| ① Claude 加载 SKILL.md | 读取 frontmatter，知道这个 skill 需要哪些包 |
| ② Claude 执行 skill 指令 | 如果指令**实际需要使用**这些包（如 Python 脚本中 import） |
| ③ Claude 自动 pip install | 在运行脚本前执行 `pip install <包>` |
| ④ 包已装则跳过 | 已有缓存时不会重复安装 |

#### ⚠️ 注意事项

1. **AI 驱动，不是 CLI 自动执行** — `dependencies` 只是元数据，告诉 Claude"这个 skill 需要什么"。Claude 会根据指令判断是否真的需要安装。不会像 `package.json` 那样一加载就自动 npm/pip install
2. **skill 正文要明确使用** — 如果正文只写"检查包是否已安装"，Claude 可能跳过安装。要让 Claude 真正用这些包干活
3. **同时支持 npm** — 官方说也支持 JavaScript npm 包，实测环境未验证
4. **requirements.txt 仍然可用** — 可以在 skill 目录放 `requirements.txt`，skill 正文引用它手动安装。`dependencies` 字段是更简洁的替代方案
5. **按需安装，不是全部安装** — 实测列出 `[tqdm, rich]`，正文只要求用 tqdm，结果只装了 tqdm（v4.67.3），rich 没装。所以列表里列了的包如果正文没用到，Claude 不会装

#### 完整示例

```yaml
---
name: slide-draft-from-csv
description: 清洗 CSV 数据，生成摘要并草拟演示文稿幻灯片
dependencies:
  - python>=3.10
  - pandas==2.2.2
  - matplotlib>=3.8
  - jinja2
---

## 使用说明

1. 读取参数 `$ARGUMENTS[0]` 指定的 CSV 文件
2. 用 pandas 分析数据，生成统计摘要
3. 用 matplotlib 生成图表
4. 用 jinja2 渲染 HTML 幻灯片
```

Claude 读到 `dependencies` 后，在执行第 2~4 步时会自动安装对应的 Python 包。

### 2.9 开放标准字段（来源: agentskills.io）

| 字段 | 类型 | 必备 | 来源 | 实测 | 说明 |
|------|------|------|------|------|------|
| `license` | string | 可选 | 🟢 开放标准 | ✅ 元数据 | 许可证标识，如 `MIT`、`Apache 2.0` |
| `compatibility` | string | 可选 | 🟢 开放标准 | ✅ 元数据 | 兼容性说明，最长500字符 |
| `metadata` | object | 可选 | 🟢 开放标准 | ✅ 元数据 | 自定义元数据，原样保留。`version` 建议放这里 |

写法：
```yaml
license: MIT
compatibility: Requires Python 3.10+, pandas>=1.5.0
metadata:
  version: 1.0.0
  author: your-name
```

### 2.10 社区使用字段

> 以下字段在官方文档和开放标准中均未确认，使用前请自行测试。

| 字段 | 类型 | 实测 | 建议 |
|------|------|------|------|
| `version` | string | YAML 有效但非规范字段 | 建议用 `metadata.version` |
| `permissionMode` | string | ✅ 实测 `bypassPermissions` 有效 | 这是 agent 字段（`.claude/agents/*.md`），SKILL.md 中能用可能是 fork 子代理继承所致。不建议写进 skill |

> `when_to_use` 和 `arguments` 已确认为官方字段，见上方对应章节。
> `license`、`compatibility`、`metadata` 来自 agentskills.io 开放标准。

---

## 三、附录

### 3.1 三级加载与支持文件

官方文档（来源: code.claude.com/docs/en/skills）：

> *"The SKILL.md contains the main instructions and is required. Other files are optional. Reference these files from your SKILL.md so Claude knows what they contain and when to load them."*

| 级别 | 文件 | 加载时机 | 用途 |
|------|------|----------|------|
| Level 1 | SKILL.md 的 YAML frontmatter | 会话启动始终加载 | 元数据 |
| Level 2 | SKILL.md 的 Markdown 正文 | skill 触发时加载 | 指令体 |
| Level 3 | `reference.md`、`scripts/` 等 | 被 SKILL.md 显式引用时才加载 | 参考文档、脚本、模板 |

官方确认的支持文件：

| 文件 | 加载方式 | 官方依据 | 说明 |
|------|---------|---------|------|
| `reference.md` | **loaded when needed**（被引用时加载→阅读） | 官方目录结构 | 普通 Markdown，Claude 读到引用时主动读取内容 |
| `examples.md` | **loaded when needed**（同上） | 官方示例中提及 | 使用示例，同上 |
| `scripts/` | **executed, not loaded**（被引用时执行→不读入上下文） | 官方目录结构+代码示例 | 可执行脚本（py/sh/js等），用 `${CLAUDE_SKILL_DIR}` 引用路径 |

引用方式（来源: 官方文档）：
```markdown
## 参考
- 详细API文档见 [reference.md](reference.md)
- 使用示例见 [examples.md](examples.md)
- 运行脚本：python3 ${CLAUDE_SKILL_DIR}/scripts/tool.py
```

> `FORMS.md` 这个文件名**不是官方标准**。官方说的是 *"templates for Claude to fill in"*（模板文件），但没有指定文件名。你可以用 `templates/` 目录或自己命名。

官方目录结构示例（来源: code.claude.com/docs/en/skills）：

```
skill-name/
├── SKILL.md              # 必须
├── reference.md           # 可选（详细参考文档，需要时加载）
└── scripts/               # 可选（可执行脚本）
    └── tool.py
```

### 3.2 YAML 逻辑值（红色加粗高亮）

```
true  false  yes  no  on  off  null
```

### 3.3 模型/可选值（紫色高亮）

```
haiku  sonnet  opus                     # model 取值
fork                                     # context 取值
general-purpose  Explore  Plan           # agent 取值
low  medium  high  xhigh  max            # effort 取值
```

### 3.4 正文标记词（橙色加粗高亮）

```
NOTE  TIP  WARNING  IMPORTANT  TODO  FIXME  HACK  XXX
```

### 3.5 FORMS.md 模板字段（暗青高亮）

> ⚠️ 以下为推荐写法，非官方标准。FORMS.md 是自由格式 Markdown。

| 字段 | 说明 |
|------|------|
| `Template` | 模板定义开始标记 |
| `Report` | 报告区块 |
| `Summary` | 摘要/总结部分 |
| `Table` | 结构化数据表格 |
| `Field` | 单个字段定义 |
| `Type` | 字段类型（string / int / boolean） |
| `Required` | 是否必填标记 |
| `Default` | 字段默认值 |
| `Severity` | 问题严重程度 |
| `TopN` | 优先修复列表 |

### 3.6 完整 SKILL.md 结构

```
╔═══════════════════════════════════════════╗
║  ───                                    ║
║  name: my-skill              # string   ║
║  description: ...             # string   ║
║  argument-hint: [...]         # string   ║
║  user-invocable: true         # boolean  ║
║  disable-model-invocation: false # boolean║
║  context: fork                # string   ║
║  agent: general-purpose       # string   ║
║  model: sonnet                # string   ║
║  effort: medium               # string   ║
║  allowed-tools:               # list     ║
║    - Read                                ║
║    - Bash(git:*)                         ║
║  hooks:                       # object   ║
║    PreToolUse: [...]                     ║
║  paths: "src/**/*.ts"         # string   ║
║  shell: bash                  # string   ║
║  license: MIT                 # string   ║
║  compatibility: ...           # string   ║
║  metadata: {}                 # object   ║
║  ───                                    ║
║  # Markdown 正文（Level 2，触发时加载）  ║
╚═══════════════════════════════════════════╝
```

### 3.7 推荐模板

```yaml
---
name: my-skill
description: 简短说清做什么和什么时候用 | 多行用 | 符号

argument-hint: "[参数提示]"

user-invocable: true
disable-model-invocation: false

context: fork
agent: general-purpose
model: sonnet
effort: medium
# permissionMode: default

allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git *)

# hooks:
#   PreToolUse:
#     - matcher: "Bash"
#       type: command
#       command: echo "start..."
#       once: true

# paths: "src/**/*.ts"
# shell: bash

# when_to_use: 触发词
# license: MIT
# compatibility: Claude Code v2.0+
# metadata:
#   version: 1.0.0
---
```

#### Markdown 正文（展示支持文件引用方式）

```markdown
<!-- 正文从这里开始，自由 Markdown，无固定格式要求 -->
<!-- 以下是支持文件的引用方式示例 -->

## 参考

- 详细API文档见 [reference.md](reference.md)
- 使用示例见 [examples.md](examples.md)
- 输出模板见 [templates/FORMS.md](templates/FORMS.md)
- 运行脚本：python3 ${CLAUDE_SKILL_DIR}/scripts/tool.py
```
