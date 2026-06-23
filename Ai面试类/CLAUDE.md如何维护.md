# CLAUDE.md 如何维护



面试常被问到："CLAUDE.md 你是怎么维护的？" 只会答 `/init` 远远不够。下面是这篇文章的核心总结。

## 1. CLAUDE.md 是什么

- 不是 README、不是注释、不是文档，而是 Claude Code **每次启动时自动读取的持久化指令文件**。
- 相当于给 Claude 写的"入职须知"：技术栈、代码风格、测试方式、踩坑清单。
- `/init` 只是冷启动：扫描 `package.json`、`Makefile`、`README` 等生成基础版，仅仅是起点。
- 小贴士：为兼容 Codex，可让 CLAUDE.md 加载 AGENTS.md，避免维护两份规则。

## 2. 四层加载体系

| 层级 | 路径 | 用途 |
|---|---|---|
| 全局 | `~/.claude/CLAUDE.md` | 个人偏好，所有项目生效 |
| 项目 | `CLAUDE.md` 或 `.claude/CLAUDE.md` | 提交到 git，团队共享 |
| 本地覆盖 | `CLAUDE.local.md` | 加入 `.gitignore`，只在本地生效 |
| 子目录 | 子目录下的 `CLAUDE.md` | **按需加载**，monorepo 友好 |

加载顺序：从文件系统根目录一路走到工作目录，内容**拼接**在一起；越靠近工作目录，优先级越高（后加载覆盖前面）。源码逻辑在 `src/utils/claudemd.ts` 的 `getMemoryFiles()` 中，先收集路径再 `dirs.reverse()`，所以 `CLAUDE.local.md` 天然覆盖项目规则。

**第一原则：不要让不同层级的 CLAUDE.md 互相矛盾。**

## 3. LLM 的指令预算

Anthropic 官方提醒：CLAUDE.md 太长，Claude 会忽略一半，重要规则会被噪音淹没。

arXiv 论文《How Many Instructions Can LLMs Follow at Once?》（2507.11538）测试结论：

- 即便最强模型，在 **500 条指令密度下准确率也只有 68%**。
- 指令越多，遵循率越低；模型会**系统性偏向序列前面的指令**。
- 瓶颈不是上下文窗口装不下，而是**模型的注意力分配不过来**。

Claude Code 系统提示本身已占用大量指令位，留给 CLAUDE.md 的有效空间没有想象中多。

**判断一条指令是否要写入 CLAUDE.md，问两个问题：**
1. 不写这条，Claude 会不会搞错？能从代码推断的，不要写。
2. 是不是每次会话都需要？只在特定场景需要的，放到 `rules/` 做路径限定。

**官方建议：像维护代码一样维护 CLAUDE.md。** 定期 review：Claude 没遵守的加重语气，本来就做对的果断删掉。

## 4. 什么样的规则真正生效

**好规则的三个特征：**
1. 一句话能写完。
2. Claude 靠自己推断不出来（如"`mvn clean package` 默认跳过测试"、"改命令入口需同步 4 个地方"）。
3. 有明确的行动指导（如"PathGuard 限制在项目根目录，禁止绝对路径逃逸"，而非"注意安全"）。

**反例：** "使用 Java 17"（`pom.xml` 里就有）、"遵循分层架构"（目录结构已说明）、"保持代码整洁"（等于没说）。

## 5. Anthropic 自己怎么写

参考 `claude-code-action` 仓库的 CLAUDE.md 结构：

- **Commands**：构建、测试、lint 命令
- **What This Is**：项目是什么，一句话
- **How It Runs**：运行机制（改代码前必须知道的事）
- **Key Concepts**：3-5 个核心要点
- **Things That Will Bite You**：踩坑清单
- **Code Conventions**：只写和默认不一样的部分

## 6. `.claude/rules/` 目录

单文件臃肿时，按主题拆分到 rules 目录。带 `paths` 前置字段的 rules 文件**只在 Claude 操作匹配路径时才加载**。

```markdown
---
paths:
  - "src/components/**/*.tsx"
  - "src/hooks/**/*.ts"
---
- 组件用函数式写法，不用 class
- props 在函数签名中解构
- 自定义 hook 以 use 开头
- 状态管理用 zustand，不用 redux
```

推荐结构：

```
.claude/
├── CLAUDE.md              # 核心规则，控制在 80 行以内
└── rules/
    ├── code-style.md      # 无路径限定
    ├── testing.md
    ├── security.md
    ├── frontend.md        # paths: ["src/**/*.tsx"]
    └── api.md             # paths: ["src/api/**/*.ts"]
```

进阶用法：用 `@path` 语法导入外部文件，如 `@README.md` 会在启动时展开注入上下文。

## 7. `/init` 与 `/memory` 的分工

- **`/init`**：冷启动，扫描仓库生成基础 CLAUDE.md。
- **`/memory`**：热更新。每个项目在 `~/.claude/projects/` 下都有 `MEMORY.md`，跨会话经验自动写入，下次启动前 200 行自动加载。

**该写在哪里？**
- CLAUDE.md：团队共享的、长期稳定的规则（提交 git）。
- memory：个人的、会变化的、日常协作积累的经验。

**节奏：** 第一周跑 `/init`，日常让 memory 自动积累；第二周开始 review memory，把通用经验提炼到 CLAUDE.md，清理过时条目。

## 8. 配置体系的四个角色

| 角色 | 职责 |
|---|---|
| **CLAUDE.md** | 管"建议" |
| **settings.json** | 管"强制"——权限、环境变量、MCP 配置，无商量余地 |
| **hooks** | 管"自动化"——必须每次执行的（如编辑后跑 Prettier），由 harness 执行 |
| **rules/** | 管"精准投放"——按路径限定加载，节省指令预算 |

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git *)"],
    "deny": ["Bash(rm -rf *)", "Bash(git push --force)"]
  }
}
```

## 9. 可直接复用的模板

```markdown
# CLAUDE.md

## Commands
- 构建：mvn clean package -DskipTests
- 测试：mvn test
- 单个测试：mvn test -Dtest=XxxTest
- 代码检查：mvn spotbugs:check
- 格式化：mvn spotless:apply

## What This Is
一句话说清楚项目是什么。

## Architecture
- 入口：Main.java → CliCommandParser 分发命令
- Agent 循环：AgentLoop.java，工具注册在 ToolRegistry
- 不要动 agent/core/ 下的接口定义，下游工具全部依赖它们

## Things That Will Bite You
- search_code 是 RAG 辅助，优先用 glob → grep → read
- 改命令入口 → 同步 Main.java + CliCommandParser + 测试 + 文档
- 测试里 API Key 全部 mock，禁止提交真实 Key

## Code Conventions
- 日志用 SLF4J，不用 System.out
- 异常不要吞掉，至少 log.warn
- public API 返回统一的 Result 包装类

## Don't
- 不要 new Thread，用 ExecutorService
- 不要往 CLAUDE.md 里加"保持代码整洁"这种废话
```

全文不到 50 行，覆盖全部关键信息，没有一句废话。

## 10. 面试三句话回答

1. **讲机制**：CLAUDE.md 是 Claude Code 的持久化指令文件，启动时自动加载。四层加载体系（系统/全局/项目/本地/子目录），越靠近工作目录优先级越高。
2. **讲原则**：arXiv 论文实测 500 条指令密度下最强模型只有 68% 准确率，所以核心规则放 CLAUDE.md 控制在 80 行内；按场景拆分到 `.claude/rules/`，用 `paths` 做路径限定按需加载。
3. **讲实践**：`/init` 冷启动生成基础版，`/memory` 自动积累跨会话经验。只写 Claude 推断不出来的东西，定期 review，像维护代码一样维护 CLAUDE.md。
