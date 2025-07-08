# 第三阶段：Go 后端与区块链交互

本文档将重点介绍如何利用你的 Go 开发经验与区块链进行后端集成。

### 章节

1.  [**`go-ethereum` (Geth) 库**](./3-1_go_ethereum_library.md)
    *   介绍 `go-ethereum` 的核心功能和模块
    *   如何在 Go 项目中引入和配置该库

2.  [**钱包、密钥与交易签名**](./3-1.5_wallet_keys_and_signing.md)
    *   在代码层面理解私钥、公钥和地址的生成过程
    *   学习如何使用私钥对交易进行签名

3.  [**Go 后端交互实践**](./3-2_go_backend_interaction.md)
    *   连接到以太坊节点
    *   查询链上数据
    *   监听链上事件
    *   构建和发送交易
    *   后端钱包管理

4.  [**dApp 架构与前后端协作**](./3-3_dapp_architecture.md)
    *   了解前端 Ethers.js/Viem 的作用
    *   设计链下服务与链上合约通信的 API
    *   dApp 的典型架构模式 