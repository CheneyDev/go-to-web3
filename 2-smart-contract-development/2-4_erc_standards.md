# 2.4: 重要的合约标准 (ERC)

以太坊的强大之处在于其可组合性——不同的智能合约可以像乐高积木一样相互交互。这种互操作性的基石是 **ERC (Ethereum Request for Comment)** 标准。ERC 是以太坊社区采纳的应用级标准，定义了一系列通用接口，使得钱包、交易所、DApp 等应用可以与不同开发者编写的合约进行统一的交互。

本章将重点介绍两个最著名、最重要的 ERC 标准：ERC-20 (同质化代币) 和 ERC-721 (非同质化代币/NFT)。

## 1. ERC-20: 同质化代币标准

ERC-20 是用于发行 **同质化代币 (Fungible Tokens)** 的标准。所谓"同质化"，意味着每个代币都是相同且可互换的。例如，你钱包里的一个 USDT 和我钱包里的一个 USDT 是完全一样的，没有区别，可以一比一交换。我们常见的各种"山寨币"、稳定币（如 USDT, USDC）以及治理代币（如 UNI, AAVE）都遵循 ERC-20 标准。

### 1.1 ERC-20 核心接口

一个标准的 ERC-20 合约必须实现以下函数和事件：

```solidity
// 函数
function name() public view returns (string) // 代币名称，如 "Tether"
function symbol() public view returns (string) // 代币符号，如 "USDT"
function decimals() public view returns (uint8) // 代币精度，通常是 18
function totalSupply() public view returns (uint256) // 总供应量
function balanceOf(address _owner) public view returns (uint256 balance) // 查询某个地址的余额
function transfer(address _to, uint256 _value) public returns (bool success) // 发送代币
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) // 从某个地址发送代币（需授权）
function approve(address _spender, uint256 _value) public returns (bool success) // 授权给另一个地址可动用的额度
function allowance(address _owner, address _spender) public view returns (uint256 remaining) // 查询授权额度

// 事件
event Transfer(address indexed _from, address indexed _to, uint256 _value);
event Approval(address indexed _owner, address indexed _spender, uint256 _value);
```

### 1.2 `transfer` vs `approve` + `transferFrom`

这是 ERC-20 中一个非常重要的设计模式：

- **`transfer(to, value)`**: 直接调用此函数，可以将代币从你自己的账户（`msg.sender`）发送到 `to` 地址。
- **`approve(spender, value)` + `transferFrom(from, to, value)`**: 这是一个两步操作，常用于智能合约交互。
    1.  你（`owner`）调用 `approve` 函数，授权给另一个地址（`spender`，通常是一个 DApp 的合约地址）可以从你的账户中动用 `value` 数量的代币。
    2.  当 `spender` 合约需要使用你的代币时（例如，在去中心化交易所中进行兑换），它会调用 `transferFrom` 函数，将代币从你的账户（`from`）转移到目标地址（`to`）。

这种模式避免了将你的代币直接发送到一个你可能不完全信任的合约中，增强了安全性。

## 2. ERC-721: 非同质化代币 (NFT) 标准

ERC-721 是用于发行 **非同质化代币 (Non-Fungible Tokens, NFT)** 的标准。与 ERC-20 不同，每个 ERC-721 代币都是独一无二、不可互换的。每个代币都有一个唯一的 `tokenId`。这使得它们非常适合用来代表独特的数字或实物资产的所有权，例如：数字艺术品、游戏道具、域名、活动门票、房产证书等。

### 2.1 ERC-721 核心接口

ERC-721 的核心功能围绕着所有权（ownership）和转移（transfer）展开。

```solidity
// 函数
function ownerOf(uint256 _tokenId) public view returns (address) // 查询某个 NFT 的所有者
function balanceOf(address _owner) public view returns (uint256) // 查询某个地址拥有的 NFT 数量
function transferFrom(address _from, address _to, uint256 _tokenId) public // 转移 NFT
function approve(address _approved, uint256 _tokenId) public // 授权单个 NFT 的操作权限
function setApprovalForAll(address _operator, bool _approved) public // 授权给某个地址操作你所有 NFT 的权限
function getApproved(uint256 _tokenId) public view returns (address) // 查询单个 NFT 的授权地址
function isApprovedForAll(address _owner, address _operator) public view returns (bool) // 查询是否已授权所有 NFT

// 事件
event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);
event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);
```

### 2.2 元数据 (Metadata) 扩展

一个 NFT 的价值通常在于它所关联的"元数据"（例如，一张 JPG 图片的 URL，或游戏道具的属性）。ERC-721 通过一个可选的元数据扩展接口 `ERC721Metadata` 来实现这一点。

```solidity
function name() external view returns (string _name); // NFT 集合的名称，如 "CryptoPunks"
function symbol() external view returns (string _symbol); // NFT 集合的符号，如 "PUNK"
function tokenURI(uint256 _tokenId) external view returns (string); // 返回一个指向 JSON 文件的 URL，其中包含了该 NFT 的详细元数据
```

`tokenURI` 是 NFT 的灵魂所在。它返回的链接通常指向一个遵循特定格式的 JSON 文件，内容如下：

```json
{
  "name": "My Awesome NFT #1",
  "description": "This is a very unique and awesome NFT.",
  "image": "https://example.com/images/1.png",
  "attributes": [
    {
      "trait_type": "Color",
      "value": "Blue"
    },
    {
      "trait_type": "Power",
      "value": "Super Strong"
    }
  ]
}
```
像 OpenSea 这样的 NFT 市场就是通过解析这个 JSON 文件来展示 NFT 的图片和属性的。

---

**总结**: ERC-20 和 ERC-721 是以太坊生态的基石。理解它们的工作原理，对于开发与代币或 NFT 相关的任何 DApp 都至关重要。幸运的是，我们通常不需要从零开始实现这些标准，[OpenZeppelin](https://www.openzeppelin.com/contracts) 提供了一系列经过安全审计的、模块化的标准合约实现，我们可以直接继承和使用。 