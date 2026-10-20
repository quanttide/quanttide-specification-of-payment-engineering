# 代金券规范

本规范定义支付工程中代金券的概念、数据约束、状态机、不变式与形式化规则。金额一律以整数分为单位，币种固定为 CNY。

## 主题

- [概念](concepts.md) — 代金券、面值、发行、核销与状态等术语，以及与优惠券、余额的区别
- [数据约束](data-constraints.md) — 字段、类型与约束
- [状态机](state-machine.md) — issued、redeemed、expired 三种状态与合法转移
- [不变式](invariants.md) — 十条必须始终成立的约束
- [形式化规则](rules.md) — issue、redeem、expire 的前置与后置条件
- [未决事项](open-issues.md) — 结算顺序、发行治理与历史结算等待定问题

代金券的业务洞察见 insight 仓库 `data/insight/voucher/index.md`。
