# chinese-code-comment-skill

一组面向代码注释的 Codex skills，默认只修改注释和文档标记，不改变业务逻辑、公共 API、字符串或错误处理。四个 skill 可以单独使用，也可以使用综合入口按固定顺序完成完整处理。

## 四个 skill

| Skill | 作用 | 显式调用命令 |
| --- | --- | --- |
| 翻译 | 将已有英文或其他语言的注释翻译成中文，不新增解释 | `chinese-code-comment-translate` |
| 注释增强 | 按 why 优先规范补充类型、成员、参数、复杂语法和每条控制流路径 | `chinese-code-comment-enrich` |
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
|   |-- chinese-code-comment-enrich/SKILL.md      # why 注释增强
|   `-- chinese-code-comment-style/SKILL.md        # 编程规范整理
|-- references/multilingual-examples.md           # 综合 skill 示例
`-- skills/chinese-code-comment-enrich/references/
    `-- multilingual-examples.md                  # 增强 skill 示例
```

## 给 Codex 安装

使用 Codex 内置安装脚本分别安装四个入口。以下命令假定目标目录中还没有同名 skill：

```bash
SKILL_INSTALLER=/home/tang/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py
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
请使用 chinese-code-comment-skill，处理 src/ 目录中的 Python 和 C++ 文件：先把现有英文注释翻译成中文，再按 why 规范补齐每个类型、成员、函数参数、复杂语法和每个 if/else/for/while 路径的注释，最后按项目的注释规范整理格式。只改注释，不改变代码逻辑。
```

## 原始 skill 内容

- 综合入口：[SKILL.md](SKILL.md)
- 翻译入口：[skills/chinese-code-comment-translate/SKILL.md](skills/chinese-code-comment-translate/SKILL.md)
- 注释增强入口：[skills/chinese-code-comment-enrich/SKILL.md](skills/chinese-code-comment-enrich/SKILL.md)
- 规范整理入口：[skills/chinese-code-comment-style/SKILL.md](skills/chinese-code-comment-style/SKILL.md)
- 多语言示例：[references/multilingual-examples.md](references/multilingual-examples.md)

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
