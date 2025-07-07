# Go to Web3

欢迎来到 "Go to Web3"！这是一个为 Go 开发者量身打造的 Web3 学习教程。无论你是希望进入 Web3 领域，还是想将 Web3 技术栈与 Go 语言结合，这里都能为你提供清晰的学习路径和实用的代码示例。

本教程从区块链的基础知识讲起，逐步深入到智能合约开发、后端集成，最后覆盖到一些高级主题，旨在帮助你构建一个完整的 Web3知识体系。

## Web3 学习路线图 (针对 Go 开发者)

欢迎来到 Web3 的世界！作为一名 Go 后端开发者，你的经验在很多方面都非常有价值，特别是在理解系统架构、并发和网络方面。这份学习路线图旨在帮助你平稳地从 Go 后端过渡到 Web3 开发。

### 第一阶段：掌握核心概念 (打好地基)

这是最重要的一步，理解 Web3 的"为什么"比直接写代码更关键。

1.  **区块链基础：**
    *   **去中心化：** 理解 Web3 的核心理念，它与 Web2 的中心化模式有何根本不同。
    *   **区块链是什么：** 弄懂链式数据结构、哈希指针、不可篡改性这些基本概念。
    *   **共识机制：** 重点理解**工作量证明 (Proof of Work, PoW)** 和**权益证明 (Proof of Stake, PoS)**。

2.  **比特币 (你的起点)：**
    *   **目标：** 阅读并理解**比特币白皮书**。
    *   **核心概念：** 重点理解 **UTXO 模型**、挖矿、私钥与公钥。

3.  **以太坊与智能合约 (可编程的区块链)：**
    *   **目标：** 理解以太坊如何将区块链从一个简单的支付网络，演变成一个"世界计算机"。
    *   **核心概念：**
        *   账户模型 vs UTXO 模型。
        *   智能合约 (Smart Contracts)。
        *   EVM (Ethereum Virtual Machine)。
        *   Gas 费用。

### 第二阶段：智能合约开发实战 (链上编程)

这是你动手写代码的阶段。

1.  **学习 Solidity 语言：**
    *   以太坊生态最主流的智能合约语言。
    *   **推荐资源：** CryptoZombies, Solidity 官方文档。

2.  **掌握开发工具链：**
    *   **Hardhat (推荐初学者)：** 一个基于 JavaScript/TypeScript 的开发环境。
    *   **Foundry (进阶选择)：** 一个基于 Rust 的开发工具箱，允许用 Solidity 写测试。

3.  **编写和部署你的第一个智能合约：**
    *   **目标：** 在测试网（如 Sepolia）上成功部署一个你自己的合约。
    *   **实践：** 编写 `SimpleStorage` 合约、编写测试、获取测试币、部署到测试网。
    *   **了解合约标准：** ERC-20 (同质化代币) 和 ERC-721 (NFT)。

### 第三阶段：与区块链交互 (发挥你的 Go 优势)

这是将你的后端技能与区块链连接起来的阶段。

1.  **Go 语言集成：**
    *   **核心库：** **`go-ethereum`** (Geth 的官方 Go 实现库)。
    *   **实践：**
        *   查询链上数据。
        *   监听链上**事件 (Events)**。
        *   构建并发送交易。
        *   在后端服务中安全地管理钱包。

2.  **了解前端交互 (了解全貌)：**
    *   **核心库：** 了解 `Ethers.js` 或 `Web3.js`。
    *   **目的：** 了解前端的工作方式能让你更好地设计整体应用架构和 API。

### 第四阶段：进阶主题与生态探索

当你掌握了基础之后，可以探索这些更广阔的领域。

1.  **DeFi (去中心化金融)：**
    *   研究 Uniswap (DEX)、Aave (借贷) 等头部项目的基本原理。
2.  **Layer 2 扩容方案：**
    *   了解 Optimism, Arbitrum, zkSync 等主流方案。
3.  **安全性：**
    *   学习常见的攻击向量（如**重入攻击 `Re-entrancy`**）。
    *   学习使用 OpenZeppelin 等经过审计的安全合约库。
4.  **Cosmos 生态 (Go 开发者的乐园)：**
    *   **Cosmos SDK:** 一个用 Go 编写的、用于构建**自定义区块链**的强大框架。
    *   是 Go 开发者深入区块链底层的巨大优势领域。

## 目录

### [第一部分：核心概念](./1-core-concepts/index.md)
- [1.1: 区块链基础](./1-core-concepts/1-1_blockchain_fundamentals.md)
- [1.2: 比特币](./1-core-concepts/1-2_bitcoin.md)
- [1.3: 以太坊](./1-core-concepts/1-3_ethereum.md)
- [1.4: Web3 行业图景](./1-core-concepts/1-4_web3_industry_landscape.md)

### [第二部分：智能合约开发](./2-smart-contract-development/index.md)
- [2.1: Solidity 入门](./2-smart-contract-development/2-1_intro_to_solidity.md)
- [2.2: 开发工具链](./2-smart-contract-development/2-2_dev_toolchains.md)
- [2.3: 第一个项目：SimpleStorage](./2-smart-contract-development/2-3_first_project_simplestorage.md)
- [2.4: ERC 标准](./2-smart-contract-development/2-4_erc_standards.md)

### [第三部分：Go 后端集成](./3-backend-integration-with-go/index.md)
- [3.1: Go-Ethereum 库](./3-backend-integration-with-go/3-1_go_ethereum_library.md)
- [3.2: Go 后端交互](./3-backend-integration-with-go/3-2_go_backend_interaction.md)
- [3.3: DApp 架构](./3-backend-integration-with-go/3-3_dapp_architecture.md)

### [第四部分：高级主题](./4-advanced-topics/index.md)
- [4.1: DeFi](./4-advanced-topics/4-1_defi.md)
- [4.2: Layer2 扩容](./4-advanced-topics/4-2_layer2_scaling.md)
- [4.3: Web3 安全](./4-advanced-topics/4-3_web3_security.md)
- [4.4: Cosmos 生态](./4-advanced-topics/4-4_cosmos_ecosystem.md)

## 如何贡献

欢迎对本教程进行贡献！你可以：
- 提出问题或建议
- 修正文档中的错误
- 提交新的内容

在提交贡献之前，请先阅读我们的贡献指南（待补充）。

## 联系我们

如果你有任何问题，欢迎通过提 issue 的方式与我们交流。 