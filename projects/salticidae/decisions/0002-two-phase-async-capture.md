---
id: projects/salticidae/decisions/0002-two-phase-async-capture
status: 已采纳
date: 2026-11-20
---

# 0002. CAPTURE 拆成"同步 Prepare + 异步 StartCapture"

## 背景

上位机发来 `CAPTURE` 后，板端要做两件事：
① 校验并准备任务（参数、IB 能力、电机可用性）；② 真正跑整轮抓拍（可能几十秒）。

如果全同步：`ControlLoop` 线程被占住，TCP 收不到 `CANCEL` / 心跳，客户端会以为板端死了。
如果全异步：`PrepareTaskForCapture` 失败时，错误只能事后通过别的消息回传，客户端拿到的是"成功受理"。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 全同步 | 错误能当场回 RESPONSE | 控制通道卡死，收不到 CANCEL |
| B 全异步 | 控制通道始终通畅 | Prepare 失败无法第一时间告知 |
| C 两阶段：Prepare 同步、StartCapture detach 异步 | 两者的优点都要 | 调用顺序有硬约束，写错就出事 |

## 决定

选 C，顺序固定为：

```
PrepareTaskForCapture（同步，可能报错）
  → StartCapture（detach 异步）
```

**两个易错点**（原笔记两处认知修正）：

- 顺序**不能反**：不是 `StartCapture` 内部去调 `Prepare`（最初理解是反的）。
- `StartCapture` 必须 detach：否则连 `CANCEL` 都收不到。

## 后果

- 放弃："一个函数里做完"的简单结构，换来 `HandleControl` 里**同步串行**的两步调用，读代码时必须知道这个约定。
- 与之平行的还有 `captureWorker_`（保护 UI）与 detach 线程（保护 `ControlLoop`）——两者**正交**，不是重复设计：
  | 机制 | 保护谁 | 不做的后果 |
  |---|---|---|
  | `captureWorker_` | UI 线程 | 点抓拍后界面"未响应" 30 秒 |
  | `plcWorker_` | UI 线程 | PLC 不可达时界面冻结 |
  | `GMKBoard::StartCapture` detach | `ControlLoop` 线程 | TCP 收不到 CANCEL / 心跳 |

## 可被追问

1. 为什么 `PrepareTaskForCapture` 必须同步？——失败要立刻在 `RESPONSE` 里告诉上位机。
2. `captureWorker_` 和 detach 线程是不是重复了？——不，保护对象不同，正交。
3. 任务运行中又来 `CAPTURE` 怎么办？——`1003 busy`（`taskRunning_` 检查），拒绝而不排队。
