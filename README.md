# 🦞 Claude Code 学习仓库

> 基于 [claw-code](https://github.com/instructkr/claw-code) 源码分析，深入学习 AI Agent Harness 架构设计

## 📖 项目背景

**Claude Code** 是 Anthropic 推出的 AI 编程助手 CLI 工具。2026年3月，其源码意外泄漏，社区迅速进行了 clean-room 重写，诞生了 **claw-code** 项目。

本仓库旨在通过源码分析，学习现代 AI Agent 的核心架构设计模式。

### 什么是 Harness 架构？

Harness 架构是 AI Agent 系统的核心设计模式，负责：

- **工具编排**: 管理和调度各种工具（文件操作、Shell 执行、API 调用等）
- **运行时管理**: 会话状态、对话历史、上下文压缩
- **任务调度**: 多轮对话循环、工具调用链
- **权限控制**: 细粒度的操作权限管理

---

## 🏗️ 项目结构

```
claw-code/
├── src/                              # Python 移植版（架构映射）
│   ├── main.py                       # CLI 入口
│   ├── runtime.py                    # 运行时 shim
│   ├── tools.py                      # 工具元数据
│   ├── commands.py                   # 命令元数据
│   └── reference_data/               # JSON 快照
│       ├── tools_snapshot.json       # 184 个工具定义
│       ├── commands_snapshot.json    # 命令定义
│       └── subsystems/               # 29 个子系统定义
│
├── rust/                             # Rust 完整实现 ⭐
│   └── crates/
│       ├── rusty-claude-cli/         # 主 CLI 二进制
│       │   └── src/main.rs           # 98KB 完整实现
│       ├── runtime/                  # 运行时核心
│       │   └── src/
│       │       ├── conversation.rs   # 对话循环
│       │       ├── session.rs        # 会话管理
│       │       ├── permissions.rs    # 权限控制
│       │       └── prompt.rs         # 系统提示词
│       ├── api/                      # Anthropic API 客户端
│       ├── tools/                    # 工具实现
│       └── commands/                 # 斜杠命令
│
└── skill-workflow-analysis.md        # Skill 执行流程分析
```

---

## 🚀 快速开始

### 1. 获取代码

```bash
git clone https://github.com/instructkr/claw-code.git
cd claw-code
```

### 2. Python 版本（元数据查看）

```bash
# 查看项目摘要
python3 -m src.main summary

# 列出所有工具
python3 -m src.main tools --limit 20

# 列出所有命令
python3 -m src.main commands --limit 20

# 一致性审计
python3 -m src.main parity-audit
```

### 3. Rust 版本（完整功能）

```bash
cd rust

# 构建
cargo build --release -p rusty-claude-cli

# 运行 REPL
cargo run -p rusty-claude-cli --

# 单次提问
cargo run -p rusty-claude-cli -- prompt "分析这个项目的架构"

# 查看帮助
cargo run -p rusty-claude-cli -- --help
```

---

## 📚 学习路径

### 初级：理解整体架构

| 序号 | 主题 | 文件 | 说明 |
|------|------|------|------|
| 1 | 项目概述 | `README.md` | 了解项目背景 |
| 2 | 数据模型 | `src/models.py` | 核心数据结构 |
| 3 | CLI 入口 | `src/main.py` | 命令行接口 |
| 4 | 工具清单 | `src/reference_data/tools_snapshot.json` | 184 个工具定义 |

### 中级：深入核心模块

| 序号 | 主题 | 文件 | 说明 |
|------|------|------|------|
| 5 | Rust 入口 | `rust/crates/rusty-claude-cli/src/main.rs` | 98KB 完整实现 |
| 6 | 对话运行时 | `rust/crates/runtime/src/conversation.rs` | 核心对话循环 |
| 7 | 工具执行 | `rust/crates/tools/src/lib.rs` | 工具路由与执行 |
| 8 | API 客户端 | `rust/crates/api/src/client.rs` | Anthropic API |

### 高级：专题深入

| 序号 | 主题 | 说明 |
|------|------|------|
| 9 | [Skill 执行流程](./skill-workflow-analysis.md) | 完整调用链路分析 |
| 10 | 权限系统 | `runtime/src/permissions.rs` |
| 11 | 会话压缩 | `runtime/src/compact.rs` |
| 12 | MCP 协议 | `runtime/src/mcp*.rs` |

---

## 🔧 核心概念

### 1. 工具系统

Claude Code 内置了 **184 个工具**，包括：

| 类别 | 工具示例 | 说明 |
|------|----------|------|
| 文件操作 | `read_file`, `write_file`, `edit_file` | 读写编辑文件 |
| Shell | `bash`, `PowerShell` | 执行命令 |
| 搜索 | `glob_search`, `grep_search` | 文件搜索 |
| 网络 | `WebFetch`, `WebSearch` | 网络请求 |
| Agent | `Agent`, `Skill` | 子代理和技能 |
| 任务 | `TodoWrite` | 任务管理 |

### 2. 斜杠命令

REPL 模式下的交互命令：

```
/help        显示帮助
/status      显示会话状态
/compact     压缩对话历史
/model       切换模型
/permissions 设置权限
/cost        显示 token 用量
/memory      查看记忆文件
/init        初始化项目
/diff        显示 git diff
/version     显示版本
/export      导出对话
/session     会话管理
/exit        退出
```

### 3. 权限模式

| 模式 | 说明 |
|------|------|
| `read-only` | 只读，不修改文件 |
| `workspace-write` | 可写工作区文件 |
| `danger-full-access` | 完全访问权限 |

---

## 📊 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        用户终端                                   │
│                          ↓                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    LineEditor                            │    │
│  │                  (监听输入)                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          ↓                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   REPL 主循环                            │    │
│  │              (区分命令/普通文本)                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          ↓                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              ConversationRuntime                         │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │    │
│  │  │   Session   │  │  ApiClient  │  │ToolExecutor │      │    │
│  │  │  (会话管理)  │  │ (API调用)   │  │ (工具执行)   │      │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          ↓                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Tools Layer                            │    │
│  │   Bash  File  Edit  Web  Agent  Skill  Todo  ...        │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 推荐目录结构

如果你要建立自己的学习仓库，建议结构如下：

```
my-claude-code-learning/
├── README.md                    # 学习笔记索引
├── notes/                       # 学习笔记
│   ├── 01-architecture.md       # 架构分析
│   ├── 02-tools.md              # 工具系统
│   ├── 03-runtime.md            # 运行时
│   └── 04-skill-workflow.md     # Skill 流程
├── code-snippets/               # 代码片段
│   └── tool-example.rs
├── diagrams/                    # 架构图
│   └── workflow.svg
└── references/                  # 参考资料
    └── links.md
```

---

## 🔗 相关资源

- [claw-code 仓库](https://github.com/instructkr/claw-code) - 本分析基于的源码
- [instructkr Discord](https://instruct.kr/) - 韩国 LLM 社区
- [oh-my-codex (OmX)](https://github.com/Yeachan-Heo/oh-my-codex) - 项目构建工具

---

## 📝 学习记录

| 日期 | 主题 | 进度 |
|------|------|------|
| - | 项目结构分析 | ✅ |
| - | Skill 执行流程 | ✅ |
| - | 待续... | 🔄 |

---

*Happy Learning! 🦞*
