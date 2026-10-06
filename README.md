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
|-- skills/
|   |-- chinese-code-comment-skill/SKILL.md       # 综合 skill
|   |-- chinese-code-comment-translate/SKILL.md   # 注释翻译
|   |-- chinese-code-comment-enrich/
|   |   |-- SKILL.md                              # why 注释增强
|   |   `-- references/multilingual-examples.md   # 增强 skill 示例
|   `-- chinese-code-comment-style/SKILL.md        # 编程规范整理
```

## Harness 自动安装协议

本节供 Codex、Claude Code、Cursor 等 Harness Agent 执行。当用户提供本仓库的 GitHub 地址（例如 `https://github.com/MrTang-Yuhan/chinese-code-comment-skill` 或其 fork）并说“安装这个 skill”“把这个仓库装上”或表达同等意图时，Agent 必须把它视为安装请求并直接完成安装。不要要求用户从 README 复制提示词、选择命令或手动搬运文件；在当前会话已有授权范围内直接执行，只有 Harness 的权限机制确实阻止写入时，才报告被阻止的目标路径并请求其标准授权。

按以下流程执行：

1. **读取并识别入口**：读取仓库根目录的 `README.md`，检查 `skills/*/SKILL.md`，识别四个可独立发现的入口：`skills/chinese-code-comment-skill/SKILL.md` 对应综合入口，以及同级的 `skills/chinese-code-comment-translate/SKILL.md`、`skills/chinese-code-comment-enrich/SKILL.md`、`skills/chinese-code-comment-style/SKILL.md` 三个子入口。默认安装四个入口；只有用户明确指定某个入口时才缩小范围。`chinese-code-comment-enrich/references/multilingual-examples.md` 是增强入口必须保留的资源。
2. **检查当前 Harness**：识别当前 Harness 的官方 skill/plugin 安装器、默认作用域、已安装目录和同名条目。优先使用 Harness 原生安装流程；不要同时通过插件和手工复制重复安装，也不要把 README 注册成 skill。若安装器支持 dry-run，先检查目标路径和冲突。
3. **执行安装**：按 Harness 默认作用域安装四个入口；用户明确指定项目级或用户级时遵循用户指定作用域。四个入口都位于 `skills/` 下的同级目录，必须分别安装到 `<skills-directory>/<skill-name>/`，每个目标目录直接包含对应的 `SKILL.md`，并保留入口目录中的相对资源。不要安装仓库根目录，也不要把四个入口套进一个额外的仓库目录；这样可以避免递归扫描时把一个入口的嵌套 `SKILL.md` 重复注册。Claude Code 使用其当前版本支持的 skills 路径（项目级通常为 `.claude/skills/<skill-name>/`，用户级通常为 `~/.claude/skills/<skill-name>/`）；本仓库没有 `.claude-plugin/plugin.json`，不应按插件 manifest 安装。
4. **处理冲突和更新**：安装与更新都以四个入口目录为单位；`<skills-directory>` 只保存已安装的 skill，不是 Git 工作树。不要把仓库克隆、解压或 `git pull` 到 `<skills-directory>` 或任何已安装 skill 的目录中，否则仓库内的其他 `SKILL.md` 会被递归扫描并造成重复。Codex 原生安装器在目标目录已存在时会拒绝覆盖；更新时应先把四个 `skills/<name>/` 入口下载到临时目录，逐项比较并保留本地修改，再同步到对应的四个目标目录。使用 Skills CLI 安装的副本应使用其 `update` 流程，不要再用另一种方式重复安装。
5. **验证并反馈**：清理旧版整仓库安装目录后，再刷新 skill 索引或重新启动会话；实际验证四个名称都可发现且每个名称只有一个路径。向用户报告使用的安装方式、实际目标路径、更新冲突和未完成项。

当 Harness 原生安装器不可用而支持 [Skills CLI](https://github.com/vercel-labs/skills) 时，Agent 可以自动执行下面的回退命令；当前 Harness 为 Claude Code 时把 `--agent codex` 改为 `--agent claude-code`，仓库链接指向 fork 时替换仓库地址，并按目标作用域增加或去掉 `--global`。四个入口位于 `skills/` 下的同级目录，`--full-depth` 用于发现这些入口。首次运行需要 Node.js 和网络访问：

```bash
npx -y skills add MrTang-Yuhan/chinese-code-comment-skill --full-depth --skill '*' --agent codex --copy --yes
```

## Agent 参考：Codex 原生安装脚本

当 Codex 原生 skill installer 可用时，由 Agent 执行以下命令分别安装四个入口。以下命令假定目标目录中还没有同名 skill：

```bash
CODEX_HOME=${CODEX_HOME:-$HOME/.codex}
SKILL_INSTALLER="$CODEX_HOME/skills/.system/skill-installer/scripts/install-skill-from-github.py"
python "$SKILL_INSTALLER" --repo MrTang-Yuhan/chinese-code-comment-skill --path skills/chinese-code-comment-skill --name chinese-code-comment-skill
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

从 GitHub 拉取仓库本身只会更新源码副本，不会自动更新 `$CODEX_HOME/skills` 中的已安装 skill；拉取后仍须按下面的更新流程同步。完成安装或更新后刷新 skill 索引或重新开始 Codex 会话。

如果此前使用旧版流程把整个仓库安装到 `chinese-code-comment-skill` 目录，请先备份并移出 `$CODEX_HOME/skills/chinese-code-comment-skill/` 及其余三个旧的顶层子 skill 目录，再按新版流程重新安装；否则旧目录中的嵌套 `skills/*/SKILL.md` 仍会被索引。不要把旧的整仓库目录和新版的四个同级目录混用。

如果 Skills CLI 的旧锁文件仍把综合入口记录为仓库根目录的 `SKILL.md`，也不要直接执行更新；先按实际作用域移除旧记录（例如 `npx -y skills remove -g chinese-code-comment-skill -y`），再使用新版四入口命令重新安装。

### 更新已有安装

更新时不要在 `$CODEX_HOME/skills` 下执行 `git clone` 或 `git pull`。对于 Codex 原生安装器，先使用临时目标目录下载四个入口，确认差异后再逐项同步；安装器本身不会覆盖已有目标：

```bash
CODEX_HOME=${CODEX_HOME:-$HOME/.codex}
SKILL_INSTALLER="$CODEX_HOME/skills/.system/skill-installer/scripts/install-skill-from-github.py"
STAGE=$(mktemp -d)
python "$SKILL_INSTALLER" --dest "$STAGE" --repo MrTang-Yuhan/chinese-code-comment-skill \
  --path skills/chinese-code-comment-skill \
         skills/chinese-code-comment-translate \
         skills/chinese-code-comment-enrich \
         skills/chinese-code-comment-style
```

逐项比较 `$STAGE/<skill-name>` 与 `$CODEX_HOME/skills/<skill-name>`，确认并保留本地修改后，再把临时目录中的四个入口同步到对应目标；不要把 `$STAGE` 或仓库目录放进 `$CODEX_HOME/skills`。如果使用 Skills CLI 安装，则只更新这四个入口：`npx -y skills update --global --yes chinese-code-comment-skill chinese-code-comment-enrich chinese-code-comment-style chinese-code-comment-translate`（项目安装把 `--global` 改为 `--project`），不要把 `skills add` 当作更新，也不要同时执行原生安装器。

原生安装器没有覆盖更新模式，不能用 `cp -r` 把新目录合并到旧目录；需要先备份并移除待更新的目标目录，再从 `$STAGE` 完整复制对应入口。全局和项目作用域也不要同时安装这四个入口，否则 Harness 可能从两个作用域发现同名 skill。

更新完成后，确认 `$CODEX_HOME/skills` 下针对本仓库只存在四个入口目录，并检查是否残留旧版嵌套入口：

```bash
find "$CODEX_HOME/skills" -type f -path '*/chinese-code-comment-*/SKILL.md' -print
find "$CODEX_HOME/skills/chinese-code-comment-skill/skills" -type f -name SKILL.md -print 2>/dev/null
```

第二条命令应没有输出；如果有输出，说明旧版整仓库目录仍在被扫描，应先备份本地修改，再将该旧目录移出 `$CODEX_HOME/skills`，最后重新加载 skill 索引。

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

- 综合入口：[skills/chinese-code-comment-skill/SKILL.md](skills/chinese-code-comment-skill/SKILL.md)
- 翻译入口：[skills/chinese-code-comment-translate/SKILL.md](skills/chinese-code-comment-translate/SKILL.md)
- 注释增强入口：[skills/chinese-code-comment-enrich/SKILL.md](skills/chinese-code-comment-enrich/SKILL.md)
- 规范整理入口：[skills/chinese-code-comment-style/SKILL.md](skills/chinese-code-comment-style/SKILL.md)
- 多语言示例：[skills/chinese-code-comment-enrich/references/multilingual-examples.md](skills/chinese-code-comment-enrich/references/multilingual-examples.md)

综合 skill 必须按翻译 → 增强 → 规范的顺序执行；只需要其中一项时，直接调用对应子 skill，避免扩大修改范围。

## 开发和校验

修改 skill 后，分别运行校验脚本：

```bash
VALIDATOR=/home/tang/.codex/skills/.system/skill-creator/scripts/quick_validate.py
python "$VALIDATOR" skills/chinese-code-comment-skill
python "$VALIDATOR" skills/chinese-code-comment-translate
python "$VALIDATOR" skills/chinese-code-comment-enrich
python "$VALIDATOR" skills/chinese-code-comment-style
git diff --check
```

修改后应按目标语言的 formatter、编译器、静态检查、文档生成器或测试继续验证。提交并推送后，GitHub 页面和 Raw URL 才会提供最新版本。
