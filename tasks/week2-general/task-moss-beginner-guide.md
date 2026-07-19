# Moss 新手教程

## 适合谁阅读

这篇教程适合刚开始接触 Moss 的 Web3 Builder，尤其是下面几类人：

- 想了解 AI Agent 如何安全调用链上协议；
- 熟悉一点 TypeScript / Node.js，但还不熟悉 Monad 生态；
- 会用钱包和 DApp，但没有深入写过协议 Adapter；
- 想从 Moss 项目入手学习开源项目结构和贡献方式。

如果你还没有读过 Moss，可以先记住一句话：Moss 是一个帮助 AI Agent 在 Monad 上构造和模拟未签名交易的框架。它不会替你签名，也不会替你发送交易。

## 学习目标

读完这篇教程后，你应该能完成以下事情：

- 理解 Moss 的四步工作流：`discover -> load -> action -> simulate`；
- 在本地安装和构建 Moss；
- 跑通官方示例；
- 知道 MCP Server 怎么接入 AI Agent；
- 知道新手应该从哪些代码和文档开始读；
- 明白 Moss 当前的安全边界和使用风险。

## 先理解 Moss 的核心概念

学习 Moss 不要一开始就钻源码，先把几个词理解清楚。

| 概念 | 新手解释 |
|------|----------|
| Agent | 帮用户理解意图、拆解任务、调用工具的 AI 程序 |
| Protocol package | 某个协议的适配包，负责 ABI、地址、参数规则和交易构造 |
| Capability | Moss 暴露给 Agent 的可执行能力，例如 transfer、approve、swap |
| Query | 只读能力，例如 balanceOf、allowance、quote |
| Action | 根据参数构造一次操作，写操作会返回未签名交易 |
| Simulation | 在链上状态上模拟交易执行，提前观察可能发生的结果 |
| Receipt | 对模拟结果的可读解释，例如“转出多少 MON，收到多少 USDC” |

最重要的是区分三件事：

- Moss 负责构造交易；
- Moss 负责模拟和解释交易效果；
- 钱包和用户负责最终签名。

## 准备环境

根据 Moss 仓库说明，本地运行需要：

- Node.js 22 或以上；
- pnpm；
- Git；
- 一个可访问 Monad RPC 的网络环境。

可以先检查本地版本：

```bash
node -v
pnpm -v
git --version
```

如果 Node 版本低于 22，建议先升级 Node。Moss 是 TypeScript monorepo，对 Node 和 pnpm 版本比较敏感，版本不对时可能会出现依赖安装或构建错误。

## 克隆项目

```bash
git clone https://github.com/nishuzumi/moss
cd moss
```

进入项目后，先看根目录：

```bash
ls
```

新手重点关注这些文件和目录：

```text
README.md
SECURITY.md
docs/
examples/
packages/
package.json
pnpm-workspace.yaml
```

不要急着一口气读完所有源码。Moss 是 monorepo，先把主线跑通，再逐步展开。

## 安装依赖和构建

```bash
pnpm install
pnpm build
```

如果一切正常，说明项目已经可以在本地编译。

如果遇到依赖或版本错误，优先检查：

- Node 是否为 22 或以上；
- 是否使用 pnpm 而不是 npm / yarn；
- 当前目录是否是 Moss 仓库根目录；
- 网络是否能访问 npm registry。

## 跑通官方示例

Moss 提供了 example，用来展示最基础的调用流程。可以先跑简单流程：

```bash
pnpm --filter @themoss/example-simple-flow wrap
```

也可以尝试 swap 示例：

```bash
pnpm --filter @themoss/example-simple-flow swap
```

这些示例的重点不是让你真实交易，而是观察 Moss 如何构造未签名交易、如何模拟、如何输出可检查的结果。

运行示例时建议重点看三类输出：

- action 阶段返回了什么交易结构；
- simulate 阶段是否成功；
- Receipt 或 Warning 里描述了什么链上变化。

如果示例因为 RPC 或 trace 能力失败，不代表代码一定有问题。Moss 的模拟依赖 Monad RPC 支持相关调试能力，有些 RPC 可能不支持完整 trace。

## 理解四步工作流

### 第一步：discover

`discover` 用来问 Moss：“现在有哪些能力可以用？”

例如 Agent 可能会发现：

- `erc.transfer`
- `erc.approve`
- `erc.balanceOf`
- `system.wrap`
- `kuru.swap`
- `kuru.quote`

这一步解决的是“我能做什么”。

### 第二步：load

`load` 用来加载某个能力的详细说明。

例如加载 `kuru.swap` 后，Agent 应该知道：

- 需要哪些参数；
- 参数类型是什么；
- 哪些字段是地址；
- 哪些字段是 token；
- 是否涉及滑点；
- 是否有风险标签。

这一步解决的是“这个能力应该怎么调用”。

### 第三步：action

`action` 用来真正构造操作。

如果是 Query，例如 `balanceOf`，它会返回读取结果。

如果是写操作，例如 `transfer` 或 `swap`，它会返回未签名交易树。这里要特别注意：Agent 不应该手工改 action 返回的交易，也不应该随意重排交易顺序。如果参数错了，正确做法是重新调用 action。

这一步解决的是“根据用户意图生成什么交易”。

### 第四步：simulate

`simulate` 用来模拟 action 返回的未签名交易。

模拟后，Moss 会把链上变化解释成 Receipt。Agent 应该拿 Receipt 和用户原始意图对比。

例如用户想把 1 MON 换成 USDC，Agent 至少要检查：

- 输入资产是不是 MON；
- 输入数量是不是 1；
- 输出资产是不是 USDC；
- 接收地址是不是用户指定地址；
- 滑点或最小收到数量是否符合要求；
- 是否出现了额外 approve 或未知转账。

这一步解决的是“这笔交易实际会造成什么结果”。

## 作为 MCP Server 使用

Moss 的一个重要使用方式是 MCP Server。MCP 可以理解成 AI Agent 调用外部工具的标准协议。

Moss MCP Server 会暴露四个工具：

```text
discover
load
action
simulate
```

构建完成后，可以在支持 MCP 的客户端中配置 Moss server。README 中的思路是运行构建后的 MCP Server CLI，并设置 `MOSS_RPC_URL`。

一个典型配置思路如下：

```json
{
  "mcpServers": {
    "moss": {
      "command": "node",
      "args": ["/path/to/moss/packages/mcp-server/dist/cli.js"],
      "env": {
        "MOSS_RPC_URL": "https://rpc.monad.xyz"
      }
    }
  }
}
```

实际路径要替换成你本地 Moss 仓库的位置。

配置好后，Agent 就可以通过 MCP 调用 Moss。这样 Agent 不需要自己知道 Kuru 的 ABI，也不需要自己拼 ERC-20 calldata，而是通过 Moss 的标准工具完成调用。

## 作为 SDK 使用

如果你想把 Moss 集成到自己的 TypeScript 项目中，可以把它当 SDK 使用。

一个简化后的思路是：

```ts
// 伪代码，仅用于理解流程
const runtime = createMonadRuntime({ rpcUrl });
const registry = createRegistry(runtime);

registry.register(systemPackage);
registry.register(ercPackage);
registry.register(kuruPackage);

const action = await registry.action("kuru", "swap", {
  account,
  tokenIn: "MON",
  tokenOut: "USDC",
  amount: "1"
});

const simulation = await simulator.simulate(action);

if (simulation.halted || simulation.warnings.length > 0) {
  throw new Error("Simulation needs review");
}

// 通过钱包展示未签名交易，由用户确认签名
```

真实代码要以 Moss 示例和当前 API 为准。新手不要直接复制伪代码进生产项目，应该先跑通官方 example。

## 新手读源码路线

Moss 的代码不少，新手可以按下面顺序读。

### 1. 先读 README 和 SECURITY

先明确 Moss 做什么、不做什么。

重点问题：

- Moss 为什么不签名、不发送？
- 什么是未签名交易？
- 模拟能保证什么，不能保证什么？
- Protocol package 为什么是可信边界？

### 2. 再读 docs/mcp-tools.md

这个文件会帮助你理解 Agent 视角下的 Moss。

重点问题：

- `discover` 输入输出是什么？
- `load` 返回哪些 schema？
- `action` 返回的交易树为什么不能手工修改？
- `simulate` 的 Warning 应该怎么处理？

### 3. 跑 examples

不要只读代码。先运行 example，再回头看源码，会容易很多。

重点观察：

- 示例注册了哪些包；
- action 如何被调用；
- simulate 如何被调用；
- 输出结果如何被检查。

### 4. 读 packages/core

`core` 是 Moss 的抽象中心。

重点关注：

- Registry 如何注册协议；
- Capability 如何描述；
- 参数 schema 如何定义；
- Receipt 覆盖验证如何工作。

不需要第一遍就完全读懂所有类型体操，先抓住调用链。

### 5. 读 packages/system 和 packages/erc

这两个包比较适合作为入门 adapter 参考。

重点关注：

- WMON wrap / unwrap 如何构造；
- ERC-20 transfer / approve 如何定义；
- balanceOf / allowance 这种 Query 如何实现；
- Receipt parser 如何解释链上事件。

### 6. 最后读 protocol-kuru

Kuru swap 会更接近真实 DeFi 协议交互，复杂度比 ERC-20 高。

重点关注：

- quote 和 swap 如何区分；
- tokenIn / tokenOut / amount 如何处理；
- swap 相关 Receipt 如何解释；
- 是否涉及 MON 和 WMON 的转换。

## 常见问题

### Moss 会帮我发交易吗？

不会。Moss 只构造和模拟未签名交易，不负责签名和发送。

### Moss 能保证交易一定成功吗？

不能。模拟是在某个链状态快照上执行的，真实上链时状态可能已经变化。Moss 可以帮助提前发现问题，但不能替代钱包确认和风险控制。

### Moss 是只支持 Monad 吗？

当前项目主要面向 Monad，README 中的目标链是 Monad mainnet，chain ID 为 `143`。

### Agent 能不能修改 action 返回的交易？

不应该。Moss 的安全模型依赖 action 返回的 Capability tree 和后续 simulate 的一致性。如果参数错了，应该重新调用 action。

### RPC 报错怎么办？

先确认 RPC 是否可用，以及是否支持 Moss 模拟需要的 trace 能力。某些免费 RPC 可能不支持完整调试接口。

### 新手适合贡献什么？

比较适合从文档、示例、FAQ、小型协议 Adapter 入手。不要一开始就改 core，因为 core 涉及 Moss 的安全边界。

## 一个推荐的一天学习计划

如果你只有一天时间，可以这样安排：

| 时间 | 任务 |
|------|------|
| 30 分钟 | 读 README，理解 Moss 定位 |
| 30 分钟 | 读 SECURITY，理解安全边界 |
| 40 分钟 | 安装依赖并运行 `pnpm build` |
| 40 分钟 | 跑 example-simple-flow |
| 60 分钟 | 读 docs/mcp-tools.md |
| 60 分钟 | 读 packages/system 和 packages/erc |
| 30 分钟 | 写一篇自己的学习笔记 |

这个顺序可以帮助你先形成整体地图，再进入细节。

## 新手练习任务

### 练习 1：解释四步流程

用自己的话解释：

```text
discover -> load -> action -> simulate
```

要求每一步都能举一个例子。

### 练习 2：分析一次 swap

假设用户说：

```text
帮我把 1 MON 换成 USDC
```

写出 Agent 应该检查的内容：

- 输入 token；
- 输入数量；
- 输出 token；
- 接收地址；
- 滑点限制；
- Receipt 中是否出现额外资产转移。

### 练习 3：读一个简单 Adapter

阅读 WMON 的 wrap / unwrap 实现，回答：

- 它需要哪些参数？
- 它构造了什么交易？
- 它如何解释 Receipt？
- 哪些地方和 ERC-20 transfer 不一样？

### 练习 4：写一个贡献计划

选择一个简单方向：

- 改进中文文档；
- 补充 Quick Start；
- 增加 FAQ；
- 研究一个 Monad 协议是否适合接入 Moss；
- 参考 WMON 写一个小型 Adapter 设计方案。

目标不是马上提交 PR，而是训练自己像开源贡献者一样理解项目。

## 学习 Moss 时最重要的心法

学习 Moss 不要只把它当作“调用链上协议的工具”。它真正值得学习的是安全分工：

- Agent 不直接掌控底层交易细节；
- 协议包提供专业的交易构造；
- 模拟器提供执行前验证；
- Receipt 把链上变化翻译成人能理解的结果；
- 用户和钱包保留最终签名权。

这套思想对 Web3 x AI Agent 很重要。未来 Agent 进入链上金融场景，真正关键的不是让它更大胆地自动执行，而是让它在执行前有更强的验证、解释和约束能力。

## 总结

Moss 的入门路径可以概括为：

```text
先理解安全边界 -> 跑通示例 -> 理解四步流程 -> 阅读简单 Adapter -> 再尝试贡献
```

对于新手来说，最好的切入点不是马上写复杂协议，而是先跑通 example，理解 MCP 工具和 Receipt 验证，再从 WMON、ERC-20 这种简单能力开始读源码。

等你能清楚解释“为什么 Moss 不签名、不发送，但仍然很有价值”时，就说明你已经抓住了 Moss 的核心。

## 参考链接

- Moss GitHub: https://github.com/nishuzumi/moss
- Moss README: https://github.com/nishuzumi/moss
- Moss MCP 工具文档: https://github.com/nishuzumi/moss/blob/main/docs/mcp-tools.md
- Moss 安全文档: https://github.com/nishuzumi/moss/blob/main/SECURITY.md
