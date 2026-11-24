---
id: projects/salticidae/decisions/0003-five-transport-modes
status: 已采纳
date: 2026-11-20
---

# 0003. 一条 PLC 链路拆成五个收发入口

## 背景

`GMKBoardWindow.cpp` 的 `makeMotor` 是全工程**唯一**的 `PlcTcpTransport` 实例化点（被调两次：EL / MMC），
但它对外暴露了五个不同用途的入口。为什么不能只留一个 `operator()`？

采集期间每秒一次的 `CHK` 状态轮询会和状态流抢同一条 socket——状态流里的 `3`/`4` 被 `CHK` 的应答"抢走"，
导致应答错位、任务提前 `done`。**加锁解决不了，只会卡死 UI。**

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 一个 `transport_` + 一把锁 | 结构最简单 | `CHK` 应答与状态流 `3`/`4` 抢 socket；加锁 → UI 卡死 |
| B 每个用途独立 socket | 彻底隔离 | PLC 侧连接数激增，现场不可控 |
| C 同一条 socket，按用途拆五个入口 + `streaming_` 标志 | 隔离语义、不增连接 | 调用方必须知道"现在该用哪个入口" |

## 决定

选 C，五个入口各解决一个问题：

| 入口 | 方法 | 解决的问题 |
|---|---|---|
| `transport_` | `operator()` | 普通请求-应答；`streaming_` 时主动拒绝 |
| `streamTransport_` | `StreamLines()` | 一请求多应答；接收时**不持锁**，给急停让路 |
| `sendOnly_` | `SendOnly()` | 流运行期间急停必须能送达，只 `::send` |
| `abortTransport_` | `AbortStream()` | 只置标志位，流自己 drain/close，避免 RST 丢掉刚发的 STOP |
| `reconnectHook_` | `Reconnect()` | 看门狗强制断链，下次命令重建 |

并发保护的四条规则：

| 场景 | 处理 |
|---|---|
| 流运行中普通命令 | `if (streaming_.load()) return {};` |
| 流运行中第二个流 | `streaming_.exchange(true)` 失败返回 false |
| 流运行中急停 | `SendOnly` 放行 |
| 状态轮询 | 调用方先 `IsStreaming()` 跳过（双保险） |

## 后果

- 放弃："一个函数搞定所有 PLC 交互"的可读性，换来**调用点必须选对入口**的纪律。
- `AbortStream` 只置标志位不直接 close，是为了防止 RST 把刚发出去的 `STOP` 丢掉——这是电力现场的真实教训。
- 急停**不参与排队**（`TryLockPlcSequence` 抢不到就拒，但 `SendOnly` 放行）。

## 可被追问

1. 为什么要五个？合成一个加锁行不行？——`CHK` 抢走 `3`/`4`；加锁 → UI 卡死。
2. 为什么 `StreamLines` 接收时不持锁？——给急停让路。
3. 急停为什么走 `SendOnly` 而不是普通命令？——普通命令在流运行期间被主动拒绝。
