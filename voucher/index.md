# 代金券规范

本文件定义支付工程中代金券的概念、数据约束、状态机与形式化规则。金额一律以整数分为单位，币种固定为 CNY。

## 概念

- 代金券（Voucher）：由公司发行、记载固定面值的内部预付凭证。它本身就是钱，可抵减应付金额，不能兑回现金。
- 面值（amount）：代金券可抵减的最大金额，整数分。
- 发行（issuance）：由已声明的发行来源创建一张代金券，形成公司负债。
- 发行来源（source）：代金券产生的原因或劳动类型，采用开放枚举。
- 持有（holding）：代金券归属某个账户。
- 核销（redemption）：在订单结算时使用代金券抵减应付金额，减少公司负债。
- 过期（expiration）：超过有效期后代金券失效。
- 适用范围（scope）：代金券可核销的业务范围，分为全场、指定业务、指定商品。
- 状态（status）：已发放（issued）、已核销（redeemed）、已过期（expired）。

与相邻概念的区别：优惠券是一条抵扣规则，按比例或门槛计算，本身不是钱；代金券是固定面值的钱。余额是账户资金的投影，代金券是独立的钱，二者在结算中的先后由计费规则决定。

## 数据约束

| 字段 | 类型 | 约束 |
|:--|:--|:--|
| id | string | 全局唯一，创建后不可变 |
| accountId | string | 非空，持券账户 |
| amount | int64 | 大于 0，单位分 |
| scope | object | type 为 all、business、product 之一；非 all 时必须给出 refs |
| source | string | 非空，开放枚举；未知来源保留原值，不得归并为 UNKNOWN |
| status | enum | issued、redeemed、expired |
| issuedAt | timestamp | 发行时间 |
| expiresAt | timestamp | 不得早于 issuedAt |
| redeemedAt | timestamp | 仅当 status 为 redeemed 时存在 |

金额为整数分，禁止浮点。每次发行对应唯一 id，一张券只对应一次发行事件。

## 状态机

状态集合为 issued、redeemed、expired。允许的转移只有三条：发行进入 issued，issued 经核销进入 redeemed，issued 经超期进入 expired。redeemed 与 expired 是终态，不得再转移。

## 不变式

1. 唯一性：券 id 全局唯一，一张券只对应一次发行。
2. 单次核销：一张券最多核销一次。
3. 不可兑回：不存在代金券到现金或余额的兑换路径。
4. 全额核销：一次核销消耗整张券，不拆分、不做部分使用；账本记录实际抵减额。
5. 不找零：面值大于应付金额时，超出部分不回退、不转为余额。
6. 过期不可用：当前时间达到或超过 expiresAt 时不得核销。
7. 范围约束：核销标的必须落在 scope 声明的范围内。
8. 账本一致：每次发行与核销各产生一条不可变交易记录；未核销且未过期的券面值之和等于账本上的代金券负债。
9. 可追溯：每张券可回溯到发行来源与核销标的。
10. 顺序确定：代金券在结算中的先后由计费规则显式给出，不由券自身决定。

## 形式化规则

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

## 未决事项

- 代金券与余额的优先顺序由计费规则给出，当前默认顺序尚待业务确认。
- 发行来源的开放枚举如何与预算挂钩，谁有权发行、发行上限多少，尚无治理规则。
- 职级价格调整后，历史预付的咨询按消费时点价还是购买时点价结算，尚未明确。
