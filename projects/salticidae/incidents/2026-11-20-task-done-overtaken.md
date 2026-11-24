---
id: projects/salticidae/incidents/2026-11-20-task-done-overtaken
date: 2026-11-20
type: 设计缺陷（已定位，机制分析）
---

# 2026-11-20 · 大图比 `TASK_DONE` 晚到，客户端统计成 0 张

## 症状

上位机收到 `TASK_DONE`（表示整轮结束），但数据通道一张图都没收到，或只收到一部分。
现场表现就是"跑完了却没图"。

## 定位

`TASK_DONE` 走控制通道（9500），图像走数据通道（9501/9502）。
**控制消息小、数据图大**，两者在不同 TCP 连接上，天然会"超车"：

> 实测 142 MB 融合图比 `TASK_DONE` 晚 15 ms 到达，客户端统计成 0 张。

## 根因

- 只看发送队列 `frames.empty()` 就判定"发完了"是错的：帧可能已被 `SenderLoop` 取出，但仍在 `send()` 里。
- 因此引入了 `inFlight` 计数（已出队、仍在写的帧也算未排空），并要求 `TASK_DONE` 前
  `WaitChannelDrained`：**队列空 + `inFlight == 0`**。

## 修法（已在实现中）

```text
SenderLoop 发完一帧 → --inFlight + notify_all()
  → 唤醒 WaitChannelDrained（不是 SenderLoop 自己，它不可能等自己）
```

完整时序还有第三个条件：

> **`TASK_DONE` 要等队列空 + `inFlight == 0` + 电机回位，三者缺一就会"客户端 0 张图"。**

`MaybeFinishTask` 必须等 `motionRunning_` 清零（PLC 还在动就不发 `TASK_DONE`）；
`EndMotionRun()` 是**唯一允许发 `TASK_DONE` 的位置**。

## 教训

- **"完成"的定义是多条件的**，不是"函数返回了"。正确性瓶颈往往就在这里。
- 跨连接的消息排序问题，加锁解决不了（那是板内），只能靠计数器 + 排空等待（板间）。
- 与 [[decisions/0003-five-transport-modes]] 同一类教训：**同一份数据走了两条通道，就必须处理乱序**。

## 关联

- [[projects/salticidae/README]] —— 第 9.7 节
- [[cpp/concurrency/condition-variable]] —— `notify_all` 的正确等待姿势
