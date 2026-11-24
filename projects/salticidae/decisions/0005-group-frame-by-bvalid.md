---
id: projects/salticidae/decisions/0005-group-frame-by-bvalid
status: 已采纳（已知缺陷未修）
date: 2026-11-20
---

# 0005. 多相机组帧靠 `bValid` 判据，不靠时间戳

## 背景

相机组（CameraGroup）把多台物理相机绑成一个逻辑设备，对外只有一个组级回调。
内部用 `GroupFrame = unordered_map<std::wstring /*SN*/, CameraFrameInfo>` 存各相机的结果，
**键是 SN，不是时间戳**——这决定了整章的推论。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 按时间戳对齐 | 理论上能容忍乱序 | 各相机时钟未必同源；还要定窗口大小 |
| B 按 SN 槽位 `bValid` 判齐 | 实现最简单，一次只有一批在飞时正确 | 没有 `frameId`，快速触发时新旧批次会混 |
| C 引入 `frameId` 严格配批 | 正确性最强 | 所有驱动都要透传 `frameId`，改动面大 |

## 决定

选 B：

```
Snap → for 循环串行 SnapCamera() → 清帧槽
   → 各相机回调填自己槽位
   → 所有槽 bValid → 整组入队 → 推送线程 CherryPick 串成 LFrame 链表
   → 用户一次回调拿到 N 张（do { ... } while (pFrame = pFrame->pNext)）
```

## 后果

- **已知缺陷（未修）**：作者 TODO 已承认——**触发快于取图时，新旧图会被拼成一组**。
- 其他已知缺陷：帧队列满时**静默丢整组**（`CameraGroup.cpp:441`）、超时残帧推送无标记（`:512`）。
- **同步保证只有 `External` 是真的**：

  | 触发模式 | 同步保证 |
  |---|---|
  | `External`（外触发） | ✅ 真硬件同步（外部 PLC 扇出） |
  | `Continues`（组内触发） | ❌ Windows 定时器 + 串行下发 |
  | `Single Frame`（软触发） | ❌ 用户调用 + 串行下发 |

- 由此区分两个常混的概念：**多相机组 ≠ GMK 景深融合**。
  融合是"同一台相机多次位移"，发生在帧被驱动交出来之前，与 CameraGroup 无关。

## 可被追问

1. 组帧"齐了"的判据是什么？——每个 SN 槽位 `bValid`，不是时间。
2. 哪种触发模式是真同步？——只有 `External`。
3. 快速触发会怎样？——无 `frameId`，新旧两批会被拼成一组（已知缺陷）。
4. 多相机组和景深融合是一回事吗？——不是，融合是单相机多次位移。
