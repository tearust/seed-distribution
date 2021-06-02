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
# seeds_account
存储 seeds 的 account. 我们叫 seeds_account.我们有私钥, 在合约初始的时候在里面 hard code 种子. 这样就省去了一个灌入种子的交易. 我们在部署合约之前会产生一个 hardcoded 的种子cid 列表.
在这个账户上 coupon 的数量也是 hardcoded, 为 10000. 也就是一共有 10000 次抽种子的机会. 
# transactions 

## 抽种子

# 以上的代币我们一共是 3 个大类
我们一共是 3 个大类代币 ABC. 每个类别都有一对 coupon 和 NFT, 对应.  A类的 coupon 只能抽取 A 类的 NFT.  当然, 他们的数量也是一一对应的. 上述的所有操作也都是分 3 次进行的. 可以认为每一个类是一个独立的合约, 但是我们希望部署在一个合约上节约费用. 操作的时候最好可以集中在一个交易完成, 只不过需要几个参数即可. 比如抽奖函数输入 A 个数, B 个数和 C 个数这 3 个参数. 代码内部分别处理各自的抽奖池子.

