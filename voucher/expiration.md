# 过期

未核销的代金券在超过有效期后进入过期状态。

## 规则

状态为 issued 且当前时间达到或超过 expiresAt 时，券转为 expired。expired 是终态，不得再核销，也不得再转移。

## 形式化规则

```text
expire(v, now) -> v'
  前置  v.status = issued 且 now >= v.expiresAt
  后置  v.status = expired
```
