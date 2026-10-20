# 数据约束

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
