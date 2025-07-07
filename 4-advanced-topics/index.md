# 第四阶段：进阶主题与生态探索

本文档将介绍 Web3 生态中的一些高级主题，帮助你扩展知识边界。

### 将包含以下内容：

- **DeFi (去中心化金融)**
  - **DEX (去中心化交易所)**：以 Uniswap V2/V3 为例，解析 AMM (自动做市商) 原理
  - **借贷协议**：以 Aave 为例，了解资金池、超额抵押和清算机制

- **Layer 2 扩容方案**
  - 为什么需要 Layer 2？(可扩展性三难困境)
  - **Optimistic Rollups**：Optimism, Arbitrum 的工作原理
  - **ZK Rollups**：zkSync, StarkNet 的基本概念和优势

- **Web3 安全**
  - **常见的智能合约漏洞**：
    - 重入攻击 (Re-entrancy)
    - 整数溢出/下溢
    - 访问控制错误
  - **安全开发最佳实践**：
    - 使用 OpenZeppelin 等经过审计的合约库
    - "检查-生效-交互"模式 (Checks-Effects-Interactions Pattern)
    - 单元测试与代码审计的重要性

- **Cosmos 生态 (Go 的世界)**
  - **Cosmos SDK 简介**：一个用于构建主权区块链的 Go 框架
  - **Tendermint 共识**：了解其 BFT 共识引擎
  - **IBC (跨链通信协议)**：Cosmos 如何实现"区块链的互联网"
  - AppChain (应用链) 的概念和优势 