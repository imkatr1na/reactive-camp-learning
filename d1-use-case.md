我选择 **Dynamic NFT Royalty System（动态 NFT 版税系统）**。https://blog.reactive.network/dynamic-nft-royalty-system/

1. 监听哪条链、哪个合约、什么事件？

   监听 Ethereum Sepolia 上的DynamicRoyaltyNFT合约产生的TokenListed事件。

2. 满足什么条件触发？

   源合约发出TokenListed，出现了被监听的上架事件，不是“发现价格合适才交易”。

3. 在哪条链做什么动作？

   STKReactive.sol将上架同步到 Reactive Kopli 上的 WrappedNFTOnKopli.sol。

4. 不用 Reactive 如何实现？简化在哪里？代价是什么？

   从 Sepolia 读取指定合约的上架事件，保存处理进度，通过目标链签名账户调用目标合约，同步上架记录，处理断线补扫、重复提交、交易失败及账户密钥管理。

   项目方可以减少自己维护后台监听器与回调发交易服务的工作。Reactive 执行需要 gas，失败回调也收费。

源链：Ethereum Sepolia
DynamicRoyaltyNFT
用户调用 list(tokenId, price)
         │
         └── 发出 TokenListed 事件
                       │
                       ▼
Reactive 事件响应层
STKReactive.sol
识别上架事件，触发对应回调
                       │
                       ▼
目标链：Reactive Kopli
WrappedNFTOnKopli
同步上架记录：mirrorListing(tokenId, price)


