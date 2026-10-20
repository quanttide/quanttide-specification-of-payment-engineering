# 发行

发行阶段创建一张代金券，记录发行来源，并形成公司负债。

## 数据

- id：全局唯一，创建后不可变
- accountId：非空，持券账户
- amount：整数分，大于 0
- source：非空，开放枚举；未知来源保留原值，不得归并为 UNKNOWN
- issuedAt：发行时间
- status：发行后为 issued

## 规则

发行来源是开放枚举，新增来源不需要修改本规范。每次发行对应唯一 id，一张券只对应一次发行事件。发行即公司负债形成，账本需要记录发行来源。

## 形式化规则

```text
issue(a, amount, source, scope, expiresAt) -> v
  前置  amount > 0 且 amount 为整数分
  后置  v.status = issued
  后置  v.issuedAt = now
  后置  形成面值为 amount 的公司负债
```
