# 形式化规则

```text
对象：券 v，账户 a，订单 o，计费规则 r
集合：Status = {issued, redeemed, expired}
函数：
  issue(a, amount, source, scope, expiresAt) -> v
  redeem(v, o, r) -> Transaction
  expire(v, now) -> v'

issue:
  前置  amount > 0 且 amount 为整数分
  后置  v.status = issued
  后置  v.issuedAt = now
  后置  形成面值为 amount 的公司负债

redeem:
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

expire:
  前置  v.status = issued 且 now >= v.expiresAt
  后置  v.status = expired
```
