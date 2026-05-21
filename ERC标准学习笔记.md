# ERC标准

## ERC-20——可替代代币标准

ERC-20代币合约用于追踪同质化代币：任何一个代笔都与其他代币完全等价；没有任何代币拥有任何特殊权利或行为。使得ERC-20代币可用作交换媒介、行使投票权、进行质押等多种用途。

### 如何创建ERC-20供应？

==固定供应==：假设需要一种总量固定为1000的代币，初始分配给部署合约的账户。

```
contract ERC20FixedSupply is ERC20 {
    constructor() ERC20("Fixed", "FIX") {
        _mint(msg.sender, 1000);
    }
}
```

这是Contracts v2的写法，不再直接操作核心变量，而是通过内部_mint函数修改供应量和余额。

==奖励矿工==：在Solidity中可以通过全局变量访问当前区块矿工的地址block.coinbase，每当有人调用代币函数，都会向该地址铸造代币奖励mintMinerReward()

```
contract ERC20WithMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {}

    function mintMinerReward() public {
        _mint(block.coinbase, 1000);
    }
}
```

==奖励自动化==：ERC20允许通过该函数扩展代币的核心功能_update，此功能也可奖励矿工

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
contract ERC20WithAutoMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {
        _mintMinerReward();
    }
    function _mintMinerReward() internal {
        _mint(block.coinbase, 1000);
    }
    function _update(address from, address to, uint256 value) internal virtual override {
        if (!(from == address(0) && to == block.coinbase)) {
            _mintMinerReward();
        }
        super._update(from, to, value);
    }
}
```

## ERC-721——NFT标准

ERC-721 是一种用于表示非同质化代币所有权的标准，也就是说，每个代币都是独一无二的。

我们将使用 ERC-721 代币来追踪游戏中的物品，每个物品都拥有独特的属性。当需要奖励玩家物品时，系统会铸造该物品并将其发送给玩家。玩家可以自由持有代币，也可以像对待区块链上的其他资产一样，根据自己的意愿与其他玩家交易。任何账户都可以调用`awardItem`铸造物品的机制。为了限制哪些账户可以铸造物品，我们可以添加访问控制功能。

```
// contracts/GameItem.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC721URIStorage, ERC721} from "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";

contract GameItem is ERC721URIStorage {
    uint256 private _nextTokenId;

    constructor() ERC721("GameItem", "ITM") {}

    function awardItem(address player, string memory tokenURI) public returns (uint256) {
        uint256 tokenId = _nextTokenId++;
        _mint(player, tokenId);
        _setTokenURI(tokenId, tokenURI);

        return tokenId;
    }
}
```

该[`ERC721URIStorage`](https://docs.openzeppelin.com/contracts/5.x/api/token/ERC721#ERC721URIStorage)合约是 ERC-721 的一个实现，它包含了元数据标准扩展IERC721Metadata以及用于存储每个令牌元数据的机制。该setTokenURI方法就来源于此：我们用它来存储项目的元数据。与ERC-20不同的是，ERC-721缺少一个decimals字段，因此每个令牌都是不同的，无法进行分区。

可以创建新物品、查询每个项目的拥有者和元数据

## ERC-1155——多代币标准(FT+NFT)

## ERC-165——接口检测标准

## ERC-223——ERC-20的改进

## ERC-827——ERC-20的扩展

## ERC-1046——可扩展元数据的代币标准

## ERC-6093——统一代币错误代码标准

## ERC-1363——可支付型代币（转账+自动调用逻辑）

## ERC-3643(前身T-REX)——合规型代币标准(常用于RWA)

