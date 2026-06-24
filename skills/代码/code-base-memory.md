# codebase-memory-mcp

> 项目地址：https://github.com/DeusData/codebase-memory-mcp
> 论文：[arXiv:2603.27277](https://arxiv.org/abs/2603.27277)

## 一句话简介

**面向 AI 编码 Agent 的代码智能引擎**：把整个仓库解析成一张持久化的"知识图谱"（函数、类、调用链、HTTP 路由、跨服务链接……），通过 MCP 协议供 Claude Code 等 Agent 查询，替代低效的 grep/read 文件循环。

## 解决的核心问题

AI Agent 探索代码库时，通常靠不断 `grep`、读文件来理解结构，**Token 消耗巨大、速度慢、还容易遗漏跨文件关系**。
codebase-memory-mcp 用预先构建好的图谱替代这种盲读：

- **5 次结构化查询 ≈ 3,400 tokens**，对比 file-by-file 探索的 ~412,000 tokens —— **节省约 99.2% Token**
- 同等任务下，**回答质量 83%、工具调用次数减少 2.1×**（论文在 31 个真实仓库上的评测）

## 核心特性

| 维度 | 亮点 |
|---|---|
| **速度** | Linux 内核（28M LOC / 75K 文件）3 分钟全量索引；Cypher 查询 <1ms |
| **语言** | 158 种语言（tree-sitter 全部内置编译进二进制） |
| **语义** | Hybrid LSP 对 Python / TS/JS / PHP / C# / Go / C / C++ / Java / Kotlin / Rust 做类型解析（媲美 IDE "Go to Definition"） |
| **部署** | 单一静态二进制（macOS / Linux / Windows），零依赖、零 Docker、零 API Key，全部本地运行 |
| **Agent 适配** | 一条 `install` 命令自动配置 11 种 Agent：Claude Code、Codex CLI、Gemini CLI、Zed、OpenCode、Antigravity、Aider、KiloCode、VS Code、OpenClaw、Kiro |
| **可视化** | 可选 UI 版本，浏览器打开 `localhost:9749` 看 3D 知识图谱 |
| **团队共享** | `.codebase-memory/graph.db.zst` 压缩快照可提交到 Git，队友克隆后免去全量重建 |

## 快速安装

**macOS / Linux：**
```bash
curl -fsSL https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.sh | bash
# 带 UI：在末尾加  -s -- --ui
```

**Windows（PowerShell）：**
```powershell
Invoke-WebRequest -Uri https://raw.githubusercontent.com/DeusData/codebase-memory-mcp/main/install.ps1 -OutFile install.ps1
.\install.ps1
```

安装后重启 Agent，对它说 **"Index this project"** 即可。

其他渠道：npm / PyPI / Homebrew / Scoop / Winget / Chocolatey / AUR / `go install`。

## 14 个 MCP 工具

**索引类**
- `index_repository`：把仓库索引进图谱（之后 watcher 自动同步）
- `list_projects` / `delete_project` / `index_status`

**查询类**
- `search_graph`：按标签、名称正则、文件、入度/出度等结构化搜索
- `trace_path`：BFS 追踪某函数的调用方与被调用方（深度 1–5）
- `detect_changes`：把 git diff 映射到受影响的符号 + 风险等级
- `query_graph`：执行只读 Cypher 查询（openCypher 子集）
- `get_graph_schema`：查看节点/边类型统计（**建议第一步先跑它**）
- `get_code_snippet`：按 qualified name 取源码
- `get_architecture`：一次返回语言、包、入口、路由、热点、聚类
- `search_code`：在已索引文件内做 grep
- `manage_adr`：架构决策记录（ADR）增删改查
- `ingest_traces`：导入运行时 trace 验证 HTTP_CALLS 边

## 图数据模型

- **节点**：`Project`、`Package`、`Folder`、`File`、`Module`、`Class`、`Function`、`Method`、`Interface`、`Enum`、`Type`、`Route`、`Resource`
- **边**：`CALLS`、`IMPORTS`、`DEFINES`、`IMPLEMENTS`、`INHERITS`、`HTTP_CALLS`、`ASYNC_CALLS`、`EMITS` / `LISTENS_ON`（Socket.IO 等通道）、`DATA_FLOWS`、`SIMILAR_TO`（MinHash 近重复）、`SEMANTICALLY_RELATED`……
- **跨仓库**：`CROSS_*` 边把多个仓库串成一张大图

## CLI 模式

每个 MCP 工具都可命令行调用：
```bash
codebase-memory-mcp cli search_graph '{"name_pattern": ".*Handler.*"}'
codebase-memory-mcp cli trace_path '{"function_name": "Search", "direction": "both"}'
codebase-memory-mcp cli query_graph '{"query": "MATCH (f:Function) RETURN f.name LIMIT 5"}'
```

## 常用配置

```bash
codebase-memory-mcp config set auto_index true          # 会话开始时自动索引新项目
codebase-memory-mcp config set auto_index_limit 50000   # 自动索引的文件数上限
codebase-memory-mcp update                              # 升级
codebase-memory-mcp uninstall                           # 卸载（不删二进制和数据库）
```

数据存放在 `~/.cache/codebase-memory-mcp/`（可用 `CBM_CACHE_DIR` 覆盖）。

## 设计哲学：不内嵌 LLM

它**只是结构分析后端**——构建并查询图谱，**不带 LLM**。
自然语言到查询的翻译，交给你正在用的 MCP 客户端（Claude Code 等）。少一份 API Key，少一份配置，少一份成本。

```
你："谁调用了 ProcessOrder？"
Agent → trace_path(function_name="ProcessOrder", direction="inbound")
codebase-memory-mcp → 返回结构化结果
Agent → 用自然语言把调用链解释给你
```
