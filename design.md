为了简化开发, 我们不打算继续兼容原来设计的 ERC20/721 标准. 而是自行开发一个很简单的合约, 只需要完成一个随机抽取种子的操作最终得到一个种子序号 和 TEA Account 的对应关系即可(当然, ETH Account在中间起到一个搭桥的作用)
 
# 每个账户存储的数据结构
```
{ 
  coupon_a:21,//u32 type, how many coupon left in this account
  coupon_b:33,
  coupon_c:90,
  seeds_a:[1, 25, 74, 21],//[seqid]. usually this could be empty before lucky draw. 
  seeds_b:[1442,2331],//[seqid]. usually this could be empty before lucky draw. 
  seeds_c:[6666,7777],//[seqid]. usually this could be empty before lucky draw. 
  tea_account:"teaaccount1234",//str, tea account
  locked: ["locked_cid_111","something"],//locked seeds, cannot be transfer, ready to send to TEA network
}
``` 
如果用户全部抽奖完毕最后应该剩下 0 个 coupon, 但是所有 coupon 都变成了 seeds.
用户可以在自己账户上看到剩余的 abc 三类的 coupon 数量和已经抽到的种子 seqid
初始情况下, 除了 seeds_account , 其他用户的账户上所有字段都是空的. 
等最后, 到了 TEA Network 开始运行之前的 Genesis block 准备阶段, 用户需要将所有种子都抽签完成并转移到 locked 中. 否则会损失机会和种子. 如何操作看下面的 transactions.
# seeds_account
```
{
 coupon_a:1000,
 coupon_b:3000,
 coupon_c:6000,
 seeds_a:[1,2,3....1000],//[seqid]. usually this could be empty before lucky draw. 
 seeds_b:[1001,1002....4000],//[seqid]. usually this could be empty before lucky draw. 
 seeds_c:[4001,4002....10000],//[seqid]. usually this could be empty before lucky draw. 
}
TEA 团队初始存储 seeds 的 account. 这个 seeds_account.我们有私钥, 我们在部署合约之前会产生一个 hardcoded 的种子cid 列表.
这个列表会出现在公司网站上, 就是一个 seqid 和种子 cid 的对应关系. 当然也可以查询每个种子的详细信息.

# transactions 
## transfer_coupon
输入参数 from, to, coupon_type_a|b|c, amount_k
任何两个账户之间转移 coupon 给另外一个. 这个交易从 from account 减去 k 个 coupon, 在 to account 增加 k 个.
k 为正整数. 
from 的账户对应的 coupon 需要余额足够.

这个 tx 主要用于从 seeds_account 向其他投资者转移 coupon. 也可以用户投资者之间互相转移. 

## 抽种子 Lucky draw
输入参数 coupon_a_amount, coupon_b_amount, coupon_c_amount
这些参数每一个都可以是 0 或者正整数. 用户的账户余额需要足够. 只要有任何 abc 一个余额不足就失败
对于 a 或者 b 或者 c 都使用如下的随机逻辑. 以下以a 为例:
从这个用户的coupon_a 减去 coupon_a_amount, 确保结果不是负数. 
循环coupon_a_account 次 {
 在 seeds_account 里面 seeds_a数组中随机抽取一个seqid, 把这个 seqid 从 seeds_account.seeds_a中移除, 加入到这个用户的seeds_a 数组中
}
如果我们的代码没有 bug, 不会发生种子数量不够抽, 或者全部 coupon 抽完还有剩余的情况发生.为了安全, 可以设定失败情况, 该交易失败eth 应该自动回滚状态.

## transfer_seeds
用户之间可以互相 transfer 种子. 
参数: seeds_type_a|b|c, [seqid], from_account, to_account

如果种子类型和 seqid 不匹配, 失败. 如果 from_account对应的类型没有这个 seqid, 失败.
如果检查通过, 从from_account 移除这个 seqid, 加入到to_account.

这种操作不太会经常发生. 所以我们每个交易只能处理一种a|b|c类型的种子. 但是每种类型可以同时转移多个 seqid

## set_tea_account
参数 tea_account
功能:填写这个用户的tea_account
允许多次运行, 每次都覆盖以前的数值. 

## transfer_to_tea
没有参数.
功能是把所有的seed_a|b|c里面的 seqid 全部转移到 locked 里面成为一个数组. 

这个操作是单向的. 如果操作, 种子就进入 locked, 没有代码可以拿出去. 

我们不会限制用户运行多少次, 反正只要用户还有 seeds 没有进入 locked 就可以运行进入, 虽然我不觉得有什么情况下需要运行多次.

# 最终统计
等达到我们和投资者约定的eth区块高度的时候. 我们会检查所有账户(除了我们自己的seeds_account)的 locked 里面的种子. 把他们输入到tea project的 genesis block 中. 这些种子的主人就是在他们账户下的tea_account.

在这个区块高度不在 locked 里面的种子, 都被认为无效. 不会引入 TEA. 以后也不再有用. 相当于投资者自行放弃



