# 核销

核销发生在订单结算时，用代金券抵减应付金额，减少公司负债。

## 前置条件

- 券状态为 issued
- 当前时间早于 expiresAt
- 核销标的落在 scope 内
- 按计费规则轮到使用代金券

## 规则

一次核销消耗整张券，不拆分、不做部分使用。面值大于应付金额时，超出部分不回退、不转为余额。账本记录实际抵减额，剩余应付按计费规则的后续工具继续结算。

一张券最多核销一次。核销后 status 为 redeemed，redeemedAt 为当前时间，并产生一条不可变交易记录。

## 形式化规则

```text
redeem(v, o, r) -> Transaction
  前置  v.status = issued
  前置  now < v.expiresAt
  前置  scopeMatch(v.scope, o)
  前置  v 在 r 的工具顺序中轮到
  设    payable = payable(o)
  设    applied = min(v.amount, payable)
  后置  v.status = redeemed
  后置  v.redeemedAt = now
  后置  账本记录 applied；v.amount 与 applied 的差额不找零、不回退
  后置  剩余应付按 r 的后续工具继续结算
```
