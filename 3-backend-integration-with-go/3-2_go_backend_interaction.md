# 3.2: Go 后端交互实践

本章是 Go 与以太坊集成的核心实践部分。我们将学习如何使用 `go-ethereum` 库执行各种常见的链上操作。

**在开始之前，请确保你有一个可用的以太坊节点 RPC URL。** 你可以从 [Infura](https://infura.io) 或 [Alchemy](https://alchemy.com) 等服务商处免费获取，也可以使用你本地运行的 Geth 节点地址（如 `http://localhost:8545`）。

## 1. 连接到以太坊节点

我们使用 `ethclient` 包来建立连接。`ethclient.Dial` 函数会创建一个客户端实例，后续所有操作都将通过这个实例进行。

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/ethereum/go-ethereum/ethclient"
)

func main() {
    client, err := ethclient.Dial("YOUR_RPC_URL")
    if err != nil {
        log.Fatalf("Failed to connect to the Ethereum client: %v", err)
    }

    // 我们可以通过查询链ID来验证连接是否成功
    chainID, err := client.ChainID(context.Background())
    if err != nil {
        log.Fatalf("Failed to get chain ID: %v", err)
    }

    fmt.Printf("Successfully connected! Chain ID: %v\n", chainID)
}
```

## 2. 查询链上数据

### 2.1 获取账户余额

`BalanceAt` 函数用于查询某个地址在特定区块高度的 ETH 余额。如果区块高度传入 `nil`，则表示查询最新区块的余额。

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"
	"github.com/ethereum/go-ethereum/params"
)

// weiToEther 将 wei 转换为 ether
func weiToEther(wei *big.Int) *big.Float {
	fBalance := new(big.Float)
	fBalance.SetString(wei.String())
	return new(big.Float).Quo(fBalance, big.NewFloat(params.Ether))
}

func main() {
    // ... (连接客户端代码)

    // 查询 Vitalik Buterin 的账户余额
    account := common.HexToAddress("0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045")
    balance, err := client.BalanceAt(context.Background(), account, nil)
    if err != nil {
        log.Fatalf("Failed to retrieve account balance: %v", err)
    }
    
    fmt.Printf("Balance in Wei: %s\n", balance.String())
    
    // 余额的单位是 Wei (1 ETH = 10^18 Wei)，需要进行转换
    ethValue := weiToEther(balance)
    fmt.Printf("Balance in Ether: %f\n", ethValue)
}
```

### 2.2 读取合约数据 (只读调用)

与智能合约交互最方便的方式是使用 `abigen` 生成 Go binding。

**第一步：准备合约和生成 Go Binding**

假设我们有上一章的 `SimpleStorage.sol` 合约。
1.  先用 `solc` 或 `hardhat` 编译合约，得到 `SimpleStorage.abi` 和 `SimpleStorage.bin` 文件。
2.  使用 `abigen` 生成 Go 文件：

    ```bash
    abigen --abi=./contracts/SimpleStorage.abi --bin=./contracts/SimpleStorage.bin --pkg=simplestorage --out=./contracts/SimpleStorage.go
    ```
    - `--abi`: ABI 文件路径。
    - `--bin`: 字节码文件路径。
    - `--pkg`: 生成的 Go 文件所属的包名。
    - `--out`: 输出的 Go 文件路径。

**第二步：在 Go 代码中调用合约**

生成的 `SimpleStorage.go` 文件中会包含 `NewSimpleStorage` 函数来创建一个合约实例，以及 `Retrieve` 方法来调用合约的同名函数。

```go
package main

import (
	"fmt"
	"log"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"
	// 导入我们刚刚生成的包
	"your_project_path/contracts" 
)

func main() {
    client, err := ethclient.Dial("YOUR_RPC_URL")
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }

    // 已部署的 SimpleStorage 合约地址
    contractAddress := common.HexToAddress("YOUR_DEPLOYED_CONTRACT_ADDRESS")
    
    // 创建一个合约实例
    instance, err := simplestorage.NewSimpleStorage(contractAddress, client)
    if err != nil {
        log.Fatalf("Failed to create contract instance: %v", err)
    }
    
    // 调用合约的 view 函数
    // 对于 view/pure 函数，最后一个参数是 bind.CallOpts，传入 nil 表示使用默认选项
    value, err := instance.Retrieve(nil)
    if err != nil {
        log.Fatalf("Failed to call retrieve function: %v", err)
    }

    fmt.Printf("Value stored in contract: %v\n", value)
}
```

## 3. 监听链上事件

监听事件对于构建响应式的 DApp 后端至关重要。我们可以使用 `abigen` 生成的 binding 来轻松地过滤和解析事件日志。

`SimpleStorage.go` 中会有一个 `FilterNumberChanged` 方法。

```go
package main

import (
	"context"
	"log"

	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/ethclient"
	"github.com/ethereum/go-ethereum/event"
	
	"your_project_path/contracts"
)

func main() {
    client, err := ethclient.Dial("YOUR_WEBSOCKET_RPC_URL") // 注意：监听事件需要 WebSocket 连接
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }

    contractAddress := common.HexToAddress("YOUR_DEPLOYED_CONTRACT_ADDRESS")
    instance, err := simplestorage.NewSimpleStorage(contractAddress, client)
    if err != nil {
        log.Fatalf("Failed to create instance: %v", err)
    }

    // 创建一个 channel 来接收事件
    logs := make(chan *simplestorage.SimpleStorageNumberChanged)
    
    // 设置过滤选项
    watchOpts := &bind.WatchOpts{
        Context: context.Background(),
        Start:   nil, // nil 表示从最新区块开始
    }
    
    // 开始订阅
    sub, err := instance.WatchNumberChanged(watchOpts, logs, nil, nil, nil)
    if err != nil {
        log.Fatalf("Failed to subscribe to event: %v", err)
    }
    defer sub.Unsubscribe()

    fmt.Println("Waiting for new events...")

    // 使用 for-select 循环来处理订阅
    for {
        select {
        case err := <-sub.Err():
            log.Printf("Subscription error: %v", err)
        case eventLog := <-logs:
            fmt.Printf("--- New Event Received ---\n")
            fmt.Printf("Changer Address: %s\n", eventLog.Changer.Hex())
            fmt.Printf("Old Number: %v\n", eventLog.OldNumber)
            fmt.Printf("New Number: %v\n", eventLog.NewNumber)
            fmt.Printf("Transaction Hash: %s\n", eventLog.Raw.TxHash.Hex())
            fmt.Printf("--------------------------\n")
        }
    }
}
```

## 4. 构建和发送交易

### 4.1 调用合约的写函数

调用写函数（会改变状态的函数）需要对交易进行签名，因此需要一个钱包的私钥。

```go
package main

import (
	"context"
	"crypto/ecdsa"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"

	"your_project_path/contracts"
)

func main() {
    client, err := ethclient.Dial("YOUR_RPC_URL")
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }

    // 1. 加载私钥
    privateKey, err := crypto.HexToECDSA("YOUR_PRIVATE_KEY")
    if err != nil {
        log.Fatalf("Failed to load private key: %v", err)
    }

    // 2. 获取公钥和地址
    publicKey := privateKey.Public()
    publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
    if !ok {
        log.Fatal("Cannot assert type: publicKey is not of type *ecdsa.PublicKey")
    }
    fromAddress := crypto.PubkeyToAddress(*publicKeyECDSA)
    
    // 3. 获取账户的 nonce
    nonce, err := client.PendingNonceAt(context.Background(), fromAddress)
    if err != nil {
        log.Fatalf("Failed to get nonce: %v", err)
    }

    // 4. 获取建议的 Gas Price
    gasPrice, err := client.SuggestGasPrice(context.Background())
    if err != nil {
        log.Fatalf("Failed to get gas price: %v", err)
    }
    
    // 5. 获取链ID
    chainID, err := client.ChainID(context.Background())
    if err != nil {
        log.Fatalf("Failed to get chain ID: %v", err)
    }

    // 6. 创建一个交易选项 (TransactOpts)
    auth, err := bind.NewKeyedTransactorWithChainID(privateKey, chainID)
    if err != nil {
        log.Fatalf("Failed to create transactor: %v", err)
    }
    auth.Nonce = big.NewInt(int64(nonce))
    auth.Value = big.NewInt(0)      // 发送的 ETH 数量，这里为 0
    auth.GasLimit = uint64(300000) // Gas 上限
    auth.GasPrice = gasPrice

    // 7. 调用合约的写函数
    contractAddress := common.HexToAddress("YOUR_DEPLOYED_CONTRACT_ADDRESS")
    instance, err := simplestorage.NewSimpleStorage(contractAddress, client)
    if err != nil {
        log.Fatalf("Failed to create instance: %v", err)
    }
    
    newValue := big.NewInt(123)
    tx, err := instance.Store(auth, newValue)
    if err != nil {
        log.Fatalf("Failed to call store function: %v", err)
    }
    
    fmt.Printf("Transaction sent! Hash: %s\n", tx.Hash().Hex())
    
    // (可选) 等待交易被打包
    receipt, err := bind.WaitMined(context.Background(), client, tx)
    if err != nil {
        log.Fatalf("Failed to wait for transaction to be mined: %v", err)
    }
    fmt.Printf("Transaction mined! Status: %v\n", receipt.Status)
}
```

### 4.2 发送原生代币 (ETH)

发送 ETH 本质上是创建一个没有 `data` 字段的交易。

```go
package main

import (
	"context"
	"crypto/ecdsa"
	"fmt"
	"log"
	"math/big"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
)

func main() {
    // ... (连接客户端, 加载私钥, 获取 nonce, gasPrice, chainID 等代码) ...
    
    // 接收方地址
    toAddress := common.HexToAddress("RECIPIENT_ADDRESS")
    // 要发送的 ETH 数量 (这里是 0.1 ETH)
    value := big.NewInt(100000000000000000) // 0.1 ETH in wei
    
    // 估算 Gas Limit
    gasLimit, err := client.EstimateGas(context.Background(), ethereum.CallMsg{
        To: &toAddress,
    })
    if err != nil {
        log.Fatalf("Failed to estimate gas: %v", err)
    }

    // 创建交易对象
    tx := types.NewTransaction(nonce, toAddress, value, gasLimit, gasPrice, nil) // data 为 nil

    // 使用私钥对交易进行签名
    signedTx, err := types.SignTx(tx, types.NewEIP155Signer(chainID), privateKey)
    if err != nil {
        log.Fatalf("Failed to sign transaction: %v", err)
    }
    
    // 发送交易
    err = client.SendTransaction(context.Background(), signedTx)
    if err != nil {
        log.Fatalf("Failed to send transaction: %v", err)
    }
    
    fmt.Printf("Transaction sent! Hash: %s\n", signedTx.Hash().Hex())
}
```

## 5. 钱包管理

`go-ethereum` 的 `accounts/keystore` 包提供了安全管理钱包密钥文件的功能。它会将私钥用你设置的密码加密后，存储在符合 Geth 规范的 JSON 文件中。

### 5.1 创建新钱包

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/ethereum/go-ethereum/accounts/keystore"
	"github.com/ethereum/go-ethereum/common/hexutil"
	"github.com/ethereum/go-ethereum/crypto"
)

func main() {
    ks := keystore.NewKeyStore("./wallets", keystore.StandardScryptN, keystore.StandardScryptP)
    password := "your-strong-password"
    
    // 创建一个新账户
    account, err := ks.NewAccount(password)
    if err != nil {
        log.Fatalf("Failed to create account: %v", err)
    }
    
    fmt.Printf("New account created!\n")
    fmt.Printf("Address: %s\n", account.Address.Hex())
    
    // 你会发现在 ./wallets 目录下多了一个 UTC--... 格式的 JSON 文件
}
```

### 5.2 导入私钥

```go
func importPrivateKey(ks *keystore.KeyStore) {
    privateKey, err := crypto.GenerateKey() // 这里用新生成的私钥做演示
    if err != nil {
        log.Fatal(err)
    }
    
    password := "your-strong-password"
    account, err := ks.Import(crypto.FromECDSA(privateKey), password, password)
    if err != nil {
        log.Fatalf("Failed to import account: %v", err)
    }
    
    fmt.Printf("Account imported! Address: %s\n", account.Address.Hex())
}
```
通过本章的学习，你已经掌握了在 Go 后端与以太坊进行交互的大部分核心技能。 