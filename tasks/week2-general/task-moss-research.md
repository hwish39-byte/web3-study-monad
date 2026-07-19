# 任务：Moss 项目研读

## 背景

Moss 是 Monad 生态上的一个开源框架，由 Nishuzumi 团队开发。它解决了一个核心问题：AI Agent 在和链上协议交互时，如何安全、准确地构造交易。

## 项目简介

> Moss turns complex DApp/protocol interactions on Monad into uniform, agent-callable capabilities — `discover → load → action → simulate`.

Moss 将 DApp / 协议的复杂交互封装成一套统一的、Agent 可调用的能力层。Agent 不需要关心 ABI、合约地址、精度计算、多 call 组装等底层细节，只需传入人类可读的参数，Moss 返回构造好的未签名交易。

## 核心问题

**AI Agent 直接构造链上交易存在严重的安全隐患：**

- Agent 需要手动解析 ABI、拼装 calldata、处理精度（decimals）、组装 multicall
- 一个"几乎正确"的交易 = 资金丢失的入口（比如 slippage 算错一位小数）
- Agent 没有"检查机制"——写了就发，错了就丢

Moss 的解决思路：**把交易构造从 Agent 手里拿过来，交给系统层做，再加上机械化的安全检查。**

## 核心能力

| 能力 | 说明 |
|------|------|
| `discover` | 发现可用的协议能力（按动词/类别筛选） |
| `load` | 加载某个能力的详情：意图、参数、风险标签 |
| `action` | 构造一笔交易（Plan），返回未签名的交易集合 + 声明的期望效果（expects） |
| `simulate` | 在实时链上状态上模拟执行 Plan，对比实际效果与声明期望，输出警告 |

**两条安全规则：**

1. **效果对账（Effects Reconciliation）** —— 模拟执行后，系统提取实际发生的资产流动（转入/转出、授权、接收方），与 Plan 中声明的 expects 对比，任何不一致即报警
2. **意图对齐（Intent Alignment）** —— Agent 将模拟结果与用户的原始意图做对比，只有 Agent 知道用户说了什么

Moss **永不签名，永不发送**。密钥留在钱包里，最终决定权在用户手上。

## 支持的协议

| 协议 | 能力 |
|------|------|
| WMON | wrap, unwrap, balanceOf |
| ERC20 / ERC721 | transfer, balanceOf, allowance, ownerOf |
| Kuru（链上 CLOB DEX）| swap（市价单, MON/USDC & MON/AUSD）, quote, markets |

## 为什么 AI Agent 需要 Moss

AI Agent 的理想工作流是："帮我用 1 MON swap 成 USDC，然后存入借贷协议"。

没有 Moss 的话，Agent 需要：
1. 找到 Kuru 的 router 地址和 ABI
2. 计算 decimals（MON=18, USDC=6）
3. 构造 swap 交易 + 可能的 wrap/unwrap + refund sweep
4. 手动模拟或直接发送 —— 风险极高

有 Moss 的话，Agent 只需要：
```
action("kuru", "swap", account, { tokenIn: "MON", tokenOut: "USDC", amount: "1" })
simulate([plan])  // 自动检查效果
```

Agent 的角色从"交易构造者"变成了"意图理解和安全审核者"——这正是 Agent 擅长的。

## 可能应用场景

- **AI 钱包助手**：用户用自然语言告诉 Agent 想做什么，Agent 通过 Moss 构造并模拟交易，确认无误后用户签名
- **DeFi 策略自动化**：多步操作（领取 → swap → 存入）的原子化组合，Agent 负责策略逻辑，Moss 负责安全执行
- **链上游戏机器人**：高频操作（下注、出牌）通过 Agent 自动执行，Moss 确保每一步都经过安全检查
- **开发者工具**：在 Foundry/Hardhat 测试脚本中用 Moss 替代手动 ABI 拼装，提升开发效率
- **跨协议聚合器**：Agent 发现最优路径（Kuru swap → 存入某借贷协议），Moss 组装多步 Plan 并逐段模拟

## 我的理解

Moss 本质上是一个 **Agent 与链上世界之间的安全抽象层**。

在 Web3 x AI Agent 这个交叉领域，最大的矛盾是：Agent 很擅长理解意图和生成代码，但**不擅长精确构造金融交易**——因为一丁点错误（精度、地址、calldata 顺序）就可能导致资金损失。Moss 把"精确构造"这件事从 Agent 手里拿过来交给系统，同时加了一道机械化的模拟验证门闸。

这个思路跟传统的"让 Agent 直接写 Solidity 并部署"不同——Moss 不是让 Agent 写合约，而是让 Agent 安全地调用已有的协议。

对我自己的启发： Agent 模块的实现思路——Agent 负责理解和决策，Moss / 合约层负责执行和验证。Web3 × AI Agent 的设计范式应该是 **Agent 做意图理解，合约/框架做安全执行**。

## 参考链接

- GitHub: https://github.com/nishuzumi/moss
- 文档: https://github.com/nishuzumi/moss/tree/main/docs
- 中文 README: https://github.com/nishuzumi/moss/blob/main/README.zh-CN.md
- Moss 在 Monad 生态中的定位: https://docs.monad.xyz

## 输出

- [x] 已阅读完整 README
- [x] 理解 Moss 的核心问题（Agent 安全构造交易）
- [x] 理解 Moss 的 4 步流程（discover → load → action → simulate）
- [x] 理解双重安全机制（效果对账 + 意图对齐）
- [x] 输出可能的应用场景分析
