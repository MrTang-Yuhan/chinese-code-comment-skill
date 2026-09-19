# chinese-code-comment-skill

为代码补充面向初学者、以 **why** 为核心的中文注释。它要求 AI 智能体覆盖类型、每个成员/结构体字段、函数和构造器的每个参数、函数契约、`if`/`for`/`while` 等控制流块，以及模板、生命周期、异步、并发等不常见语法，并保持代码逻辑不变。

## 仓库内容

```text
chinese-code-comment-skill/
|-- SKILL.md                              # Codex skill 主入口
`-- references/multilingual-examples.md   # 多语言详尽注释示例
```

## 给 Codex 安装

### 默认目录

```bash
git clone https://github.com/MrTang-Yuhan/chinese-code-comment-skill.git ~/.codex/skills/chinese-code-comment-skill
```

如果设置了 `CODEX_HOME`，安装到该目录下的 `skills`：

```bash
git clone https://github.com/MrTang-Yuhan/chinese-code-comment-skill.git "$CODEX_HOME/skills/chinese-code-comment-skill"
```

安装后应满足以下结构：

```text
<skills-directory>/chinese-code-comment-skill/SKILL.md
<skills-directory>/chinese-code-comment-skill/references/multilingual-examples.md
```

也可以在 GitHub 页面下载 ZIP，解压后把仓库根目录放到上述位置；或直接下载 Raw 文件：

<https://raw.githubusercontent.com/MrTang-Yuhan/chinese-code-comment-skill/main/SKILL.md>

只下载 `SKILL.md` 时，若需要多语言样例，还要同时下载 `references/multilingual-examples.md`，并保持相对目录结构。

### 验证和刷新

确认入口文件存在并检查 frontmatter：

```bash
test -f ~/.codex/skills/chinese-code-comment-skill/SKILL.md
sed -n '1,8p' ~/.codex/skills/chinese-code-comment-skill/SKILL.md
```

如果安装在 `CODEX_HOME`，把命令中的默认路径替换为：

```bash
test -f "$CODEX_HOME/skills/chinese-code-comment-skill/SKILL.md"
sed -n '1,8p' "$CODEX_HOME/skills/chinese-code-comment-skill/SKILL.md"
```

完成安装或更新后，刷新 Codex 的 skill 索引；如果当前会话没有刷新入口，重新开始一个 Codex 会话。显式调用名称为 `chinese-code-comment-skill`。

更新已克隆的 skill：

```bash
git -C ~/.codex/skills/chinese-code-comment-skill pull --ff-only
```

使用 `CODEX_HOME` 时：

```bash
git -C "$CODEX_HOME/skills/chinese-code-comment-skill" pull --ff-only
```

## 给其他 AI 智能体安装

1. 使用仓库地址、ZIP 或 Raw URL 下载文件。
2. 把包含 `SKILL.md` 的目录注册为该工具的 skill、agent instruction 或 system prompt 资源；不要只把 README 当作技能指令。
3. 加载完整的 `SKILL.md`。处理某种语言时，再按文档链接加载 `references/multilingual-examples.md`。
4. 刷新工具索引或重新开始 agent 会话。
5. 在任务提示中明确目标文件、语言/版本、只改注释的范围和是否允许修改业务逻辑。

推荐的调用提示：

```text
请使用 chinese-code-comment-skill，为 src/ 目录中的 C++ 代码补充面向初学者的中文注释。只修改注释，不改变代码逻辑。
```

如果工具支持显式 URL 来源，可使用：

```text
https://github.com/MrTang-Yuhan/chinese-code-comment-skill
```

如果工具只接受单文件，则使用：

```text
https://raw.githubusercontent.com/MrTang-Yuhan/chinese-code-comment-skill/main/SKILL.md
```

但单文件模式无法自动获得多语言参考，需另行提供 `references/multilingual-examples.md`。

## 开发和校验

修改 skill 后，可使用 Codex 内置 skill-creator 校验脚本：

```bash
python /home/tang/.codex/skills/.system/skill-creator/scripts/quick_validate.py .
```

该脚本检查 frontmatter、技能名称和基础结构；示例代码仍应按目标语言的格式化器、编译器或静态检查工具验证。

## 重要约束

- 注释以“为什么这样写、有什么约束、不这样写会怎样”为重点，不机械翻译代码。
- C/C++ 的结构体、类和联合体成员必须逐一注释，不能只注释类型整体。
- 函数和构造器的每个参数必须逐一解释用途、why、边界/默认值、可空性、所有权或副作用；参数名和数量要与实际签名一致。
- Python 的每个装饰器（包括参数化、堆叠、`@property`、`@classmethod`、`@staticmethod`）都要说明定义期/调用期行为、包装关系、顺序、元数据和副作用 why；其他语言的注解/Attribute/属性也要注明其真实生效时机。
- 每个控制流块和嵌套分支都要说明进入原因、处理目标及后续影响。
- 默认只修改注释，不重构或修复业务逻辑。
- 本地新增内容需要提交并推送后，GitHub 页面和 Raw URL 才会提供最新版本。
