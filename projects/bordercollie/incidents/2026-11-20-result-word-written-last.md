---
id: projects/bordercollie/incidents/2026-11-20-result-word-written-last
date: 2026-11-20
type: 协议约定（写错顺序就丢结果）
---

# 2026-11-20 · 结果字必须最后写，PLC 靠它判断"明细已就绪"

## 症状

PLC 偶尔读到半新半旧的检测结果：坐标是这一件的，结果字却是上一件的，或者明细还没写完 PLC 就读走了。

## 定位

出站路径 `ComResultSend::resultSend`（`communicationpointtopoint.cpp:988`）的写入顺序：

```text
先写 p2p_data 明细（坐标 / 数值）
→ 写条码
→ 最后写结果字 1/2
```

入站同理：读到非 0 → emit → **写回 0**（握手复位）。

## 根因

PLC 侧轮询各地址，**没有事务概念**——它读你写的寄存器，
所以"谁先可见"完全由**写入顺序**决定。结果字如果先写，PLC 立刻读到"完成了"，明细却是空的。

## 修法约定

- **结果字是唯一的"就绪信号"**，必须最后写；PLC 侧应以"结果字变化"为触发条件，而不是用定时器盲读。
- 进入同一 P2P 层的最小单位是 16 位字：

  | C++ 类型 | 占用 | 说明 |
  |---|---|---|
  | `bool` | 1 word | 写 0/1（Modbus 用 FC16 写寄存器，不是线圈） |
  | `ushort` | 1 word | 原样 |
  | `int` / `float` | 2 words | `memcpy`（x86 小端 → **低字在前**） |
  | `string` | N words | Modbus：每寄存器高字节先；MC：每 word 内交换字节 |

- float 拆成两个寄存器后**低字在前**，这是最容易和"Modbus 协议大端"混淆的一处：
  Modbus 协议本身大端（libmodbus 已处理 16 位），但 32 位值项目按小端内存布局；RTU 的 CRC 是小端。

## 教训

- 跨进程通信的"完成"是**协议约定**，不是函数返回（与 [[projects/salticidae/incidents/2026-11-20-task-done-overtaken]] 同一类教训）。
- 字节序必须按"协议层 / 值层 / 单字内"三层分别说清楚，混在一起讲必然出错。

## 关联

- [[projects/bordercollie/notes/p2p-word-ordering]] —— 编码对照表
- [[projects/bordercollie/decisions/0003-polling-inbound-address]]
