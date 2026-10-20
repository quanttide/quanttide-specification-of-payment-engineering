# 持有

发行后的代金券进入持有阶段，直到核销或过期。

## 数据

- scope：type 为 all、business、product 之一；非 all 时必须给出 refs
- expiresAt：不得早于 issuedAt
- status：issued
- redeemedAt：空

## 规则

代金券归属持券账户，面值固定，不能兑回现金或余额。适用范围与有效期在发行时确定，持有期间不得改变。
