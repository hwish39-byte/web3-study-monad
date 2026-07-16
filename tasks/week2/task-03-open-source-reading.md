# 任务：学习阅读真实开源项目

## 背景

作为 Web3 Builder，阅读和理解开源项目是核心技能。通过分析三个不同量级和定位的 Web3 项目，学习 Maintainer 如何组织代码、文档和社区。

## 目标

- 浏览项目 README，理解项目定位和核心价值
- 查看 Docs、Issues、PRs、Discussions，理解社区运作
- 分析项目目录结构，理解模块划分逻辑
- 总结优秀项目的组织共性

## 项目分析

### 项目 1：Moss（nishuzumi/moss）

| 维度 | 内容 |
|------|------|
| 定位 | Agent 与 Monad 链上协议之间的安全抽象层 |
| 语言 | TypeScript |
| ⭐ 42 / Forks 13 / Issues 28 |
| 成熟度 | Alpha 阶段，未审计 |
| 维护者 | Nishuzumi 团队 |

**README 风格：**
- 高度技术化，文档驱动
- 开篇即给出核心流程：`discover → load → action → simulate`
- 安全警告放在最前面（WARNING 块）
- 每个设计决策都有 ADR（Architecture Decision Record）

**目录结构：**

```
moss/
├── packages/
│   ├── core/                  纯逻辑，零链上数据
│   ├── simulator/             模拟验证引擎
│   ├── erc/                   ERC20/ERC721 标准 ABI 层
│   ├── system/                Monad 实例 / WMON 适配器
│   ├── protocol-*/           每个协议一个包（Kuru 等）
│   └── mcp-server/            MCP 工具（discover/load/action/simulate）
├── docs/
│   ├── getting-started.md     入门指南
│   ├── mcp-tools.md           MCP 工具参考
│   ├── protocol-onboarding.md 协议接入指南
│   ├── agent-skill.md         Agent 行为规则
│   ├── adr/                   架构决策记录
│   └── CONTEXT.md             术语表
├── examples/
│   ├── example-simple-flow/   基础流程 demo
│   └── agent-swap/            Agent 实际交易示例
└── README.md
```

**Issue / PR 特点：**
- 有 Protocol Onboarding 的 Issue 模板，降低外部贡献门槛
- 28 个 open issues，以功能请求和 Bug 为主
- PR 提交需要遵循模板

**值得学习的点：**
- 每一个包都有明确的职责和依赖边界（ADR 0006）
- ADR 记录每个设计决策及其 trade-off，方便新贡献者理解"
为什么这么设计"
- 安全警告放在 README 最显眼位置

---

### 项目 2：Hardhat（NomicFoundation/hardhat）

| 维度 | 内容 |
|------|------|
| 定位 | 以太坊专业开发环境（编译、测试、部署、调试） |
| 语言 | TypeScript + Solidity |
| ⭐ 8,492 / Forks 1,730 / Issues 619 |
| 成熟度 | 生产级，v3 开发中 |
| 维护者 | Nomic Foundation |

**README 风格：**
- 极简，一句话说明 + 安装命令
- 没有长篇大论，直接上手 `npx hardhat --init`
- 主要文档链接到官网 hardhat.org/docs

**目录结构：**

```
hardhat/
├── packages/             monorepo 多包结构
├── docs/                 文档
├── e2e/                  端到端测试
├── end-to-end/           另一个 e2e 目录？
├── scripts/              工具脚本
├── .changeset/           版本管理（Changesets）
├── .claude/              Claude 项目配置
├── img/                  图片资源
├── AGENTS.md             AI Agent 指引
├── CLAUDE.md             Claude 行为规范
├── CONTRIBUTING.md       贡献指南
├── FUNDING.json          资助信息
└── pnpm-workspace.yaml   pnpm workspace 配置
```

**Issue / PR 特点：**
- 619 个 open issues，社区活跃度极高
- Pull Requests 有严格的 CI/CD 流程
- 有 `.claude/` 和 `AGENTS.md` / `CLAUDE.md` —— 为 AI 编码助手提供项目上下文
- 使用 Changesets 管理版本发布

**值得学习的点：**
- AGENTS.md / CLAUDE.md 是新兴的最佳实践 —— 为 AI 编码工具提供项目上下文，减少"猜项目结构"
- 极简 README + 完整官网文档的分层策略 —— README 给"要不要用"的信息，文档给"怎么用"的信息
- monorepo 用 pnpm workspace + changesets 做包管理

---

### 项目 3：Solidity（argotorg/solidity）

| 维度 | 内容 |
|------|------|
| 定位 | 智能合约编程语言（编译器） |
| 语言 | C++ |
| ⭐ 25,679 / Forks 6,144 / Issues 794 |
| 成熟度 | 生产级，长期维护 |
| 维护者 | Ethereum Foundation 核心团队 |

**README 风格：**
- 经典开源项目风格：背景 → 构建 → 示例 → 文档 → 贡献
- 提供多种沟通渠道（Matrix, Gitter, Forum, X, Mastodon）
- 代码示例 + 链接到完整文档

**目录结构：**

```
solidity/
├── libevmasm/            EVM 汇编器
├── liblangutil/          语言工具（词法/语法分析基础）
├── libsmtutil/           SMT 求解工具（形式化验证）
├── libsolc/              Solidity 编译器 C 接口
├── libsolidity/          Solidity 编译器中（解析、语义分析、代码生成）
├── libsolutil/           通用工具库
├── libstdlib/            标准库
├── libyul/               Yul 中间语言
├── solc/                 命令行编译器入口
├── test/                 测试
├── docs/                 文档
├── scripts/              构建和发布脚本
├── cmake/                CMake 配置
├── tools/                辅助工具
├── deps/                 依赖
├── CMakeLists.txt        顶层构建文件
├── CODING_STYLE.md       编码规范
└── ReviewChecklist.md    Review 检查清单
```

**Issue / PR 特点：**
- 794 个 open issues，大规模社区
- 严格的项目管理和里程碑（Projects 板块）
- ReviewChecklist.md —— Review 规范化
- CODING_STYLE.md —— 统一的编码风格
- 使用 CI（CircleCI）+ Docker 构建

**值得学习的点：**
- lib* 命名规范：每个库的职责极其清晰（libsolidity 是编译器核心，libevmasm 是 EVM 汇编器）
- CODING_STYLE.md + ReviewChecklist.md —— 规模化开源项目需要文档化的流程
- Changelog.md + ReleaseChecklist.md —— 版本发布流程规范化

---

## 跨项目对比

| 维度 | Moss | Hardhat | Solidity |
|------|------|---------|----------|
| ⭐ Stars | ~42 | ~8,492 | ~25,679 |
| 阶段 | Alpha | 生产级 | 生产级（多年） |
| 语言 | TypeScript | TypeScript | C++ |
| README 风格 | 技术文档风 | 极简上手 | 经典开源 |
| 包管理 | pnpm monorepo | pnpm monorepo | CMake |
| 文档策略 | README + docs/ 目录 | 极简 README + 官网 | README + ReadTheDocs |
| AI 就绪 | 有 agent-skill.md | 有 AGENTS.md / CLAUDE.md | 无专用 AI 配置 |
| 贡献门槛 | ADR + 协议模板 | CONTRIBUTING.md + CI | CODING_STYLE + Checklist |

## 我的发现 / 总结

### 优秀开源项目的组织共性

1. **README 是项目的门面**
   - 小项目（Moss）：README = 完整文档，看完就知道能不能用、怎么用
   - 大项目（Hardhat）：README = 电梯演讲，完整信息放官网
   - 超大项目（Solidity）：README = 标准模板，重点是"如何参与"

2. **目录结构反映架构哲学**
   - Moss 的 packages/ 按协议拆分（每个协议一个包），体现插件化设计
   - Hardhat 的 monorepo 按功能分层（core / plugins / tooling）
   - Solidity 的 lib* 按编译器阶段拆分（解析 → 语义 → 代码生成）

3. **文档是产品的延伸**
   - 都有 README + 专门文档目录
   - Moss 的 ADR 记录设计决策，适合小团队知识传承
   - Solidity 的 CODING_STYLE + ReviewChecklist 适合大规模协作

4. **AI 友好正在成为新趋势**
   - Moss 有 agent-skill.md，告诉 AI Agent 如何安全使用
   - Hardhat 有 AGENTS.md / CLAUDE.md，指导 AI 编码助手
   - 未来项目可能需要考虑"如何在 AI 时代被正确理解"

### 一个感兴趣的功能点（Moss）

Moss 的 `simulate` 核心机制 —— 效果对账（Effects Reconciliation）。它通过 `debug_traceCall` 模拟交易执行，提取实际发生的资产流动（转入/转出、授权、接收方），然后与 Plan 中声明的 expects 对比。任何不一致即报警。

这个机制对 Agent 安全极其重要，但实现依赖于 Monad 节点支持 `debug_traceCall` —— 约一半的第三方免费 RPC 不支持该接口。如果能在不依赖 debug_traceCall 的情况下实现模拟验证，将大幅提升 Moss 的可用性。

## 参考链接

- Moss: https://github.com/nishuzumi/moss
- Hardhat: https://github.com/NomicFoundation/hardhat
- Solidity: https://github.com/argotorg/solidity

## 输出

- [x] 浏览三个项目 README，理解各自定位
- [x] 分析项目目录结构
- [x] 对比不同规模和阶段的项目的文档 / 组织策略
- [x] 总结优秀开源项目的共性
- [x] 找到一个感兴趣的功能点（Moss 的模拟验证机制)
