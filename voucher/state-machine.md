# 状态机

状态集合为 issued、redeemed、expired。允许的转移只有三条：发行进入 issued，issued 经核销进入 redeemed，issued 经超期进入 expired。redeemed 与 expired 是终态，不得再转移。
