# Moss 开源项目介绍

## 项目概览

Moss 是 Nishuzumi 团队开源的一个 TypeScript 框架，主要面向 Monad 生态中的 AI Agent 链上交互场景。它的核心目标是：把复杂的 DApp / 协议调用封装成一组统一、可发现、可加载、可执行、可模拟验证的能力，让 AI Agent 不需要直接手写 ABI、合约地址、calldata 和多步骤交易细节。

项目官方用一句话描述 Moss：

> Moss turns complex DApp/protocol interactions on Monad into uniform, agent-callable capabilities — `discover -> load -> action -> simulate`.

也就是说，Moss 并不是一个钱包，也不是一个自动交易机器人，而是一个夹在 AI Agent 和链上协议之间的安全抽象层。Agent 负责理解用户意图和做决策，Moss 负责把意图转换成未签名交易，并在交给钱包签名前进行模拟和结果解释。

## Moss 想解决的问题

AI Agent 很擅长理解自然语言、拆解任务和生成代码，但它并不天然适合直接构造金融交易。链上交易对精度和顺序极其敏感，一个看起来很小的错误都可能造成真实资金损失。例如：

- token decimals 计算错误，导致转账金额放大或缩小；
- 合约地址、ABI 或函数选择器填错；
- swap 交易没有正确处理滑点；
- approve 授权对象或额度不符合用户意图；
- 多步骤交易顺序错误，导致资产流向不可控；
- Agent 只知道自己“构造了交易”，却没有机制验证交易实际会造成什么效果。

Moss 的设计思路是：不要让 Agent 自己拼交易，而是让每个协议通过 Protocol package 暴露标准能力。协议包负责维护合约地址、ABI、参数规则、calldata 构造、Receipt 解析等细节；Agent 只通过统一接口调用这些能力。

这样一来，Agent 的角色从“底层交易构造者”变成“用户意图理解者 + 交易审核者”。这更符合 AI Agent 的能力边界，也更适合高风险的 Web3 场景。

## 核心工作流

Moss 的标准工作流分为四步：

| 步骤 | 作用 |
|------|------|
| `discover` | 发现当前有哪些协议能力或查询能力可用，例如 transfer、approve、swap、balanceOf |
| `load` | 加载某个能力的完整说明，包括参数 schema、字段含义、风险标签和调用约束 |
| `action` | 构造协议操作。如果是查询，返回链上读取结果；如果是写操作，返回未签名交易树 |
| `simulate` | 在 Monad 链状态上模拟未签名交易，解析事件和资产变化，输出 Receipt 和 Warning |

这个流程让 Agent 在执行链上动作前有一个明确的检查路径：先知道能做什么，再理解怎么做，然后构造交易，最后模拟验证结果。

其中最关键的是 `simulate`。Moss 不只关心交易是否能执行成功，还会提取模拟过程中产生的链上变化，并要求协议包给出完整、按顺序、不可遗漏的 Receipt 解释。Agent 和用户可以根据 Receipt 判断交易结果是否符合原始意图。

## 安全模型

Moss 的安全边界非常清晰：它只构造和模拟未签名交易，不签名、不发送、不托管私钥。最终是否签名仍然由用户的钱包决定。

它的安全模型主要包含几层：

### 1. Capability tree

每个写操作会被构造成一个有序的 Capability tree。这个树可以包含多笔直接交易，也可以表示一个复杂协议动作中的多个子步骤。Moss 要求这些步骤保持确定顺序，Agent 不应该手工修改或重排 action 返回的交易树。如果输入参数变化，就应该重新调用 `action`。

### 2. Simulation

Moss 会使用 Monad 的链状态模拟交易执行。模拟阶段会观察交易产生的事件、native MON 转账和其他可追踪变化。如果模拟失败，或者出现无法解释的风险，流程应该停止，不进入签名环节。

### 3. Receipt 覆盖验证

Moss 的一个重要设计是 Receipt 覆盖验证。协议包需要为每个 Capability 提供 Receipt parser，把原始链上 Change 解释成用户和 Agent 可以阅读的结果。

Core 会要求 Receipt 对模拟产生的 Change 做完整覆盖：

- 不能遗漏 Change；
- 不能重复解释 Change；
- 不能替换 Change；
- 不能打乱顺序；
- 不能用文本解释掩盖真实链上变化。

这使得 Moss 不只是“调用前模拟一下”，而是把模拟结果变成一份可审计的语义说明。

### 4. Intent Alignment

Moss 能验证交易实际产生了什么链上变化，但只有 Agent 知道用户最初说了什么。因此最终还需要 Agent 将 Receipt 与用户原始意图做对比。

例如用户说“用 1 MON 换 USDC，滑点最多 0.5%”，那么 Agent 需要检查 Receipt 中的输入资产、输出资产、数量、接收地址、协议和滑点限制是否都一致。

这一步叫意图对齐。Moss 提供机械化的事实依据，Agent 负责判断这些事实是否符合用户意图。

## 当前支持的能力

根据项目 README，Moss 当前目标链是 Monad mainnet，chain ID 为 `143`。目前支持的协议能力主要包括：

| 类型 | 包 | Capability | Query |
|------|------|------------|-------|
| WMON | `@themoss/system` | `wrap`、`unwrap` | `balanceOf` |
| ERC-20 / native MON | `@themoss/erc` | `transfer`、`approve` | `balanceOf`、`allowance`、`metadata` |
| ERC-721 | `@themoss/erc` | `transfer` | `ownerOf`、`balanceOf` |
| Kuru | `@themoss/protocol-kuru` | `swap` | `quote` |

从覆盖范围看，Moss 还处于早期阶段，主要支持基础 token 操作、WMON 以及 Kuru swap。它的价值不在于已经覆盖了很多协议，而在于提供了一套可扩展的协议适配框架。

未来如果更多 Monad 生态协议接入 Moss，Agent 就可以用同一套接口安全调用不同协议，而不是为每个协议单独写交易逻辑。

## 仓库结构

Moss 是一个 pnpm monorepo，主要代码放在 `packages/` 下：

```text
moss/
├── packages/
│   ├── core/          核心抽象、Registry、Capability tree、Receipt 验证
│   ├── simulator/     模拟执行和链上 Change 提取
│   ├── erc/           ERC-20 / ERC-721 标准能力
│   ├── system/        Monad runtime、WMON、系统级协议能力
│   ├── protocol-*/    具体协议适配包，例如 Kuru
│   └── mcp-server/    MCP Server，暴露 discover/load/action/simulate 工具
├── docs/              文档、MCP 工具说明、协议接入指南、ADR
├── examples/          示例项目和简单流程
└── README.md
```

这个结构很清楚地体现了 Moss 的设计哲学：核心框架和具体协议解耦，每个协议通过独立 package 接入。对于开源贡献者来说，这种结构也降低了贡献门槛，因为新增协议通常不需要改动 core，只需要参考已有 adapter 实现新的 protocol package。

## MCP Server 形态

Moss 很适合作为 MCP Server 使用。MCP Server 会把 Moss 的四个核心步骤暴露给支持 MCP 的 Agent 客户端：

- `discover`
- `load`
- `action`
- `simulate`

这意味着 Codex、Claude Desktop 或其他 MCP 客户端可以通过 Moss 调用 Monad 链上协议能力。Agent 不需要内置每个协议的 ABI 和交易构造逻辑，只需要按 MCP 工具返回的 schema 传参。

MCP 形态是 Moss 和 AI Agent 结合最自然的方式。它把 Moss 从一个普通 SDK 变成了“Agent 可调用的链上能力网关”。

## SDK 使用方式

除了 MCP Server，Moss 也可以作为 TypeScript SDK 嵌入到开发者自己的应用中。

典型流程是：

1. 创建 Monad runtime；
2. 注册 system、erc、kuru 等协议包；
3. 通过 registry 调用某个 action，例如 `kuru.swap`；
4. 使用 simulator 模拟返回的 Capability；
5. 检查是否有 halted 或 warnings；
6. 确认无误后，将未签名交易交给钱包或签名模块。

这种方式适合构建 AI 钱包助手、DeFi 自动化系统、交易确认 UI、开发者工具和协议测试脚本。

## 适合的应用场景

### AI 钱包助手

用户用自然语言表达意图，例如“帮我把 1 MON 换成 USDC”。Agent 通过 Moss 构造 swap 交易并模拟，确认 Receipt 与用户意图一致后，再交给钱包签名。

### DeFi 策略自动化

Agent 可以负责更高层的策略逻辑，例如“领取奖励、换成稳定币、再存入借贷协议”。Moss 则负责每一步协议调用的交易构造和模拟验证。

### 协议 Copilot

协议方可以提供 Moss adapter，让 Agent 安全调用自己的协议。这样用户不用理解复杂合约接口，也能通过自然语言执行协议操作。

### 开发者工具

开发者可以用 Moss 替代手写 ABI 拼装，在测试脚本或内部工具中快速构造并模拟链上操作。

## 项目成熟度和风险

Moss 当前仍处于 Alpha 阶段，README 明确提示项目未经审计，不应使用生产资金。这一点非常重要。

它目前的主要限制包括：

- 协议覆盖面还比较窄；
- 模拟依赖 RPC 能力，尤其是 trace 相关能力；
- 模拟只能代表某个链状态快照，不保证真实上链时状态不变；
- Moss 不负责签名和发送交易，钱包审核仍然不可替代；
- Protocol package 本身属于可信代码，如果协议包恶意构造交易并配套伪装解释，Moss 不声称能完全防御；
- 跨链结果不在 Moss 的验证范围内。

因此，Moss 更适合作为研究、开发和早期生态集成工具，而不是直接用于真实大额资金操作。

## 我的理解

我认为 Moss 的价值在于，它抓住了 Web3 x AI Agent 中非常关键的一点：Agent 不应该直接掌控底层交易构造。

在传统 Web2 场景里，Agent 调错一个 API 可能只是数据错误或任务失败；但在 Web3 场景里，交易一旦签名上链，就可能造成不可逆的资产损失。Web3 Agent 的核心难题不是“能不能调用链上协议”，而是“能不能在调用前证明这笔交易符合用户意图”。

Moss 给出的答案是：让协议包负责精确构造，让模拟器负责验证事实，让 Receipt 负责解释结果，让 Agent 负责意图对齐，让钱包保留最终签名权。

这个分工很合理：

- Agent 做理解和决策；
- Moss 做构造和验证；
- 协议包做专业适配；
- 钱包做签名和最终确认；
- 用户保留最终控制权。

这也是我觉得 Moss 值得研究的原因。它不是简单地把 AI 接到链上，而是在认真思考 AI 进入高风险金融执行环境时应该有哪些边界、约束和验证机制。

## 总结

Moss 是 Monad 生态中一个面向 AI Agent 的链上安全执行框架。它通过 `discover -> load -> action -> simulate` 的流程，把复杂协议调用封装为统一能力，并在签名前提供模拟验证和 Receipt 解释。

虽然它目前还处于 Alpha 阶段，协议覆盖也不多，但它的架构方向很有启发意义：未来的 Web3 Agent 不应该只是“会发交易”，而应该具备可验证、可解释、可审计的执行流程。

对于正在学习 Monad、AI Agent 和 Web3 开源贡献的 Builder 来说，Moss 是一个很好的研究对象。它既有清晰的工程结构，也有明确的安全边界，还处在早期阶段，适合通过文档、协议 adapter、示例和开发者体验优化参与贡献。

## 参考链接

- GitHub: https://github.com/nishuzumi/moss
- README: https://github.com/nishuzumi/moss
- MCP 工具文档: https://github.com/nishuzumi/moss/blob/main/docs/mcp-tools.md
- 安全文档: https://github.com/nishuzumi/moss/blob/main/SECURITY.md
