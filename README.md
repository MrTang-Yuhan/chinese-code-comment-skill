# chinese-code-comment-skill

一组面向代码注释的 Agent Skills，可在 Codex、Claude Code 等 Harness 中使用；默认只修改注释和文档标记，不改变业务逻辑、公共 API、字符串或错误处理。四个 skill 可以单独使用，也可以使用综合入口按固定顺序完成完整处理。

Harness Agent 收到本仓库链接和“安装这个 skill”的请求时，请直接执行下文的 [Harness 自动安装协议](#harness-自动安装协议)。

支持语言：C、C++、Python、JavaScript、TypeScript、Java、Go、Rust、C#。

## 四个 skill

| Skill | 作用 | 显式调用命令 |
| --- | --- | --- |
| 翻译 | 将已有英文或其他语言的注释翻译成中文，不新增解释 | `chinese-code-comment-translate` |
| 注释增强 | 按 why 优先规范补充类型、成员、参数、复杂语法和每条控制流路径；覆盖 PyTorch 张量维度变化 | `chinese-code-comment-enrich` |
| 规范整理 | 让注释符合目标语言、项目配置、formatter/linter 和文档工具规范 | `chinese-code-comment-style` |
| 综合处理 | 依次调用翻译 → 注释增强 → 规范整理 | `chinese-code-comment-skill` |

四个入口都会在开始时检查当前 Codex 会话是否有 Context7 MCP；涉及版本敏感的第三方库、框架或 API 时，优先用 Context7 核对相关文档。

在支持 `$skill-name` 显式语法的客户端中，调用形式分别是 `$chinese-code-comment-translate`、`$chinese-code-comment-enrich`、`$chinese-code-comment-style` 和 `$chinese-code-comment-skill`；也可以在自然语言提示中写出完整 skill 名称。

## 仓库结构

```text
chinese-code-comment-skill/
|-- SKILL.md                                      # 综合 skill
|-- skills/
|   |-- chinese-code-comment-translate/SKILL.md   # 注释翻译
|   |-- chinese-code-comment-enrich/
|   |   |-- SKILL.md                              # why 注释增强
|   |   `-- references/multilingual-examples.md   # 增强 skill 示例
|   `-- chinese-code-comment-style/SKILL.md        # 编程规范整理
```

## Harness 自动安装协议

本节供 Codex、Claude Code、Cursor 等 Harness Agent 执行。当用户提供本仓库的 GitHub 地址（例如 `https://github.com/MrTang-Yuhan/chinese-code-comment-skill` 或其 fork）并说“安装这个 skill”“把这个仓库装上”或表达同等意图时，Agent 必须把它视为安装请求并直接完成安装。不要要求用户从 README 复制提示词、选择命令或手动搬运文件；在当前会话已有授权范围内直接执行，只有 Harness 的权限机制确实阻止写入时，才报告被阻止的目标路径并请求其标准授权。

按以下流程执行：

1. **读取并识别入口**：读取仓库根目录的 `README.md`、`SKILL.md`，检查 `skills/*/SKILL.md`，识别四个可独立发现的入口：根目录 `SKILL.md` 对应 `chinese-code-comment-skill`，以及 `skills/chinese-code-comment-translate/SKILL.md`、`skills/chinese-code-comment-enrich/SKILL.md`、`skills/chinese-code-comment-style/SKILL.md` 对应的三个子入口。默认安装四个入口；只有用户明确指定某个入口时才缩小范围。`chinese-code-comment-enrich/references/multilingual-examples.md` 是增强入口必须保留的资源。
2. **检查当前 Harness**：识别当前 Harness 的官方 skill/plugin 安装器、默认作用域、已安装目录和同名条目。优先使用 Harness 原生安装流程；不要同时通过插件和手工复制重复安装，也不要把 README 注册成 skill。若安装器支持 dry-run，先检查目标路径和冲突。
3. **执行安装**：按 Harness 默认作用域安装四个入口；用户明确指定项目级或用户级时遵循用户指定作用域。每个目标目录必须直接包含对应的 `SKILL.md`，并保留入口目录中的相对资源；不要把四个入口套进一个额外的仓库目录。Claude Code 使用其当前版本支持的 skills 路径（项目级通常为 `.claude/skills/<skill-name>/`，用户级通常为 `~/.claude/skills/<skill-name>/`）；本仓库没有 `.claude-plugin/plugin.json`，不应按插件 manifest 安装。
4. **处理冲突**：不要覆盖已有本地修改。目标已存在时，按 Harness 的安全更新流程比较、跳过或更新，并记录冲突；不能安全处理时保留现状并向用户报告。
5. **验证并反馈**：刷新 skill 索引或重新启动会话，实际验证四个名称都可发现和调用；向用户报告使用的安装方式、实际目标路径、已安装入口和未完成项。

当 Harness 原生安装器不可用而支持 [Skills CLI](https://github.com/vercel-labs/skills) 时，Agent 可以自动执行下面的回退命令；当前 Harness 为 Claude Code 时把 `--agent codex` 改为 `--agent claude-code`，仓库链接指向 fork 时替换仓库地址，并按目标作用域增加或去掉 `--global`。由于仓库根目录已有 `SKILL.md`，`--full-depth` 用于继续发现三个子入口。首次运行需要 Node.js 和网络访问：

```bash
npx -y skills add MrTang-Yuhan/chinese-code-comment-skill --full-depth --skill '*' --agent codex --copy --yes
```

## Agent 参考：Codex 原生安装脚本

当 Codex 原生 skill installer 可用时，由 Agent 执行以下命令分别安装四个入口。以下命令假定目标目录中还没有同名 skill：

```bash
CODEX_HOME=${CODEX_HOME:-$HOME/.codex}
SKILL_INSTALLER="$CODEX_HOME/skills/.system/skill-installer/scripts/install-skill-from-github.py"
python "$SKILL_INSTALLER" --repo MrTang-Yuhan/chinese-code-comment-skill --path . --name chinese-code-comment-skill
python "$SKILL_INSTALLER" --repo MrTang-Yuhan/chinese-code-comment-skill --path skills/chinese-code-comment-translate
python "$SKILL_INSTALLER" --repo MrTang-Yuhan/chinese-code-comment-skill --path skills/chinese-code-comment-enrich
python "$SKILL_INSTALLER" --repo MrTang-Yuhan/chinese-code-comment-skill --path skills/chinese-code-comment-style
```

安装后应满足：

```text
<skills-directory>/chinese-code-comment-skill/SKILL.md
<skills-directory>/chinese-code-comment-translate/SKILL.md
<skills-directory>/chinese-code-comment-enrich/SKILL.md
<skills-directory>/chinese-code-comment-enrich/references/multilingual-examples.md
<skills-directory>/chinese-code-comment-style/SKILL.md
```

也可以从 GitHub 下载后，把四个包含 `SKILL.md` 的目录分别注册到工具的 skill 目录；不要只注册 README，也不要把四个目录再套一层同名目录。

完成安装或更新后刷新 skill 索引或重新开始 Codex 会话。当前已安装目录存在时，安装脚本会拒绝覆盖；更新时应在保留本地修改的前提下重新复制对应目录，或按工具提供的 skill 更新流程执行。

## 调用示例

### 只翻译

```text
请使用 chinese-code-comment-translate，把 src/legacy.py 中已有英文注释和 docstring 翻译成中文。
不要新增注释，也不要翻译函数名、API 名称、字符串字面量或代码示例，只修改注释文本。
```

### 只增强

```text
请使用 chinese-code-comment-enrich，为 src/queue.cpp 按 why 规范补充中文注释，逐一覆盖 class 成员、struct 字段、构造器参数以及每个分支、循环退出和异常路径。
```

### 只整理规范

```text
请使用 chinese-code-comment-style，检查 src/api.ts 的 JSDoc 是否符合项目 TypeScript 规范，修正标签、位置、换行和参数名，但保留注释含义，不改业务代码。
```

### 完整处理

```text
请使用 chinese-code-comment-skill，处理 src/ 目录中的 Python 和 C++ 文件：先把现有英文注释翻译成中文，再按 why 规范补齐每个类型、成员、函数参数、复杂语法和每个 if/else/for/while 路径的注释；如果有 PyTorch 代码，也标注所有张量操作的维度变化和形状不变关系，并在每个函数开头声明统一符号。最后按项目的注释规范整理格式。只改注释，不改变代码逻辑。
```

## 原始 skill 内容

- 综合入口：[SKILL.md](SKILL.md)
- 翻译入口：[skills/chinese-code-comment-translate/SKILL.md](skills/chinese-code-comment-translate/SKILL.md)
- 注释增强入口：[skills/chinese-code-comment-enrich/SKILL.md](skills/chinese-code-comment-enrich/SKILL.md)
- 规范整理入口：[skills/chinese-code-comment-style/SKILL.md](skills/chinese-code-comment-style/SKILL.md)
- 多语言示例：[skills/chinese-code-comment-enrich/references/multilingual-examples.md](skills/chinese-code-comment-enrich/references/multilingual-examples.md)

综合 skill 必须按翻译 → 增强 → 规范的顺序执行；只需要其中一项时，直接调用对应子 skill，避免扩大修改范围。

## 开发和校验

修改 skill 后，分别运行校验脚本：

```bash
VALIDATOR=/home/tang/.codex/skills/.system/skill-creator/scripts/quick_validate.py
python "$VALIDATOR" .
python "$VALIDATOR" skills/chinese-code-comment-translate
python "$VALIDATOR" skills/chinese-code-comment-enrich
python "$VALIDATOR" skills/chinese-code-comment-style
git diff --check
```

修改后应按目标语言的 formatter、编译器、静态检查、文档生成器或测试继续验证。提交并推送后，GitHub 页面和 Raw URL 才会提供最新版本。
