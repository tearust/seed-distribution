为了简化开发, 我们不打算继续兼容原来设计的 ERC20/721 标准. 而是自行开发一个很简单的合约, 只需要完成一个随机抽取种子的操作最终得到一个种子 CID 和 TEA Account 的对应关系即可(当然, ETH Account在中间起到一个搭桥的作用)
 
# 每个账户存储的数据结构
```
{ 
  coupon:21,//u32 type, how many coupon left in this account
  seeds:["cid111", "cid222"],//[cid]. usually this could be empty before lucky draw
  tea_account:"teaaccount1234",//str, tea account
}
``` 
最后应该剩下 0 个 coupon, 但是所有 coupon 都变成了 seeds.

我们可以在最后读取这个数据结构.

# 存储代币	
有两种 token, 一个是 ERC20 的 coupon. 每一个 coupon 表示一次抽奖机会. 
另一个代币是ERC721 的 NFT. 每一个都是一个 IPFS CID.
coupon 的数量和 NFT 数量是一样多的. 也就是说每个 coupon 抽走一个 NFT. 不会有抽不到的情况.

# 抽奖交易
这个交易的发起人的账号上需要有 Coupon. 交易参数就是 coupon 的数量. 这个发起人把自己账上的 N 个 coupon 去抽奖获得 N 个 NFT. 当然, N 不能大于他的 Coupon token 的余额.
交易执行的时候产生一个公平的随机数. 然后随机抽取一个 NFT 转移到这个用户的账上.

# 灌入初始 NFT 交易
部署合约的这个用户有权灌入初始的 NFT. 以一个 CID 数组的方式灌入. 灌入 NFT 的时候同时生成等量的 ERC20 coupon 给这个部署合约的用户.

# 其他标准的操作
比如 ERC20 代币之间的转账, ERC721 代币之间的转账. 这些都是基本操作, 不在此叙述了		

# 销毁操作
销毁的是 NFT. 具体操作方式是我们提供一个没有私钥的死地址. 用户可以把自己的 NFT 转移到这个地址. 注意这时候用户需要输入一个参数. 这个参数是用户在 TEA proejct 上的账户地址. 合约只需要记录 (该用户 ETH 地址, 该用户 TEA 地址, NFT 的 ID)这样一个 tuple 就可以了. 我们随后需要在 ETH 的链上寻找这个 tuple 的 storage 即可拿到所有的对应关系. 和这个任务无关的, 我们自己在 TEA 上mint 新的 NFT.

# 以上的代币我们一共是 3 个大类
我们一共是 3 个大类代币 ABC. 每个类别都有一对 coupon 和 NFT, 对应.  A类的 coupon 只能抽取 A 类的 NFT.  当然, 他们的数量也是一一对应的. 上述的所有操作也都是分 3 次进行的. 可以认为每一个类是一个独立的合约, 但是我们希望部署在一个合约上节约费用. 操作的时候最好可以集中在一个交易完成, 只不过需要几个参数即可. 比如抽奖函数输入 A 个数, B 个数和 C 个数这 3 个参数. 代码内部分别处理各自的抽奖池子.

