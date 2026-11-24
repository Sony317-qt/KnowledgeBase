---
id: projects/bordercollie/decisions/0003-polling-inbound-address
status: 已采纳
date: 2026-11-20
---

# 0003. 入站靠"轮询读地址 + 写回 0"握手

## 背景

PLC 与上位软件之间的点对点（P2P）通信，最小单位是 **16 位字**。触发、命令都得靠地址上的值表达。
软件既要收指令，也要回结果。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A PLC 主动推送触发 | 实时性好 | Modbus / MC / FINS 都不支持主动推送，只能轮询 |
| B 每事件一个线程 + `QTimer` 轮询读地址 | 三套协议统一实现 | 轮询周期决定延迟；事件多会串行 |
| C 单线程状态机轮询所有地址 | 资源省 | 一个慢地址拖垮全部 |

## 决定

选 B：

```text
入站：每个事件在自己的线程里 QTimer 轮询读地址
      读到非 0 → emit → 写回 0（握手复位）；读失败 → emit 断线
出站：ComResultSend::resultSend
      先写 p2p_data 明细 → 条码 → 最后写结果字 1/2
命令回复：ComAssist::on_reply（独立线程）写 reply_address
```

连接与重连：

```text
UI 连接按钮 → DialogCommunication::pushConnected()        dialogcommunication.cpp:287
            → ComPointToPoint::connectCom(ip, port)        communicationpointtopoint.cpp:69
                 ├─ com_core_->connect()（Modbus: modbus_new_tcp + modbus_connect）
                 ├─ registerEvent()：每个事件 new 对象 + 独立 QThread + QTimer（默认 200ms）
                 └─ connected_ = true

断线 → communicationDisconnected
     → reconnect_timer_.start(500ms) → reconnected() → reconnectCom() → 重新 registerEvent
```

三套 P2P 协议：

| 协议 | 实现 | 地址模型 | 默认端口 |
|---|---|---|---|
| Modbus TCP | 原生 libmodbus | 纯数字寄存器（项目里 0 基） | 502 |
| 基恩士 MC | `PointToPointComInterface` + HslCommunication(.NET) | `D100` 这类软元件（只支持 D 区） | 库决定 |
| 欧姆龙 FINS TCP | 同上 | 内存区 + 地址（`D100`/`CIO100`） | 9600 |

## 后果

- 放弃：推送式实时性。**轮询周期直接决定响应延迟和 PLC 压力**，PLC 触发脉冲必须宽于轮询周期。
- 所有事件共用一把锁 → 事件多会串行。
- 结果字**最后写**是协议级约定：PLC 应以"结果字变化"作为"明细已就绪"的信号（见 [[incidents/2026-11-20-result-word-written-last]]）。
- 换 PLC 品牌不只是改下拉框：地址表、字节序假设、.NET 依赖（`HslCommunication.dll` 等）都要跟着改。

## 可被追问

1. 入站是轮询还是推送？——轮询；周期 200 ms 可配。
2. 为什么软件读完触发字要写回 0？——握手复位，让 PLC 知道"已受理"，避免同一触发被处理两次。
3. 默认端口是多少？——**8888 ≠ Modbus 502**，第一次连不上先看这里。
4. Modbus 地址怎么填？——十进制 **0 基**，不能填 40001；从站号用 libmodbus 默认（`0xFF`，没有 `modbus_set_slave`）。
