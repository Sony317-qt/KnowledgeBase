---
id: projects/bordercollie/incidents/2026-11-20-calib-with-only-offset
date: 2026-11-20
type: 设计陷阱（使用条件苛刻的 API）
---

# 2026-11-20 · 配了 `with_source_`，`getModel()` 就只剩一个 offset

## 症状

标定做完了、配置也加载了，但算出来的物理坐标有明显偏差，或者直接失败（`applyCore` 返回失败）。

## 定位

`CalibModel` 的模型选择：配置了 `with_source_`（随动）时，`getModel()` **只算 `with.offset` 就 return**，
不再返回核心层（`internal` / `npoints` / `joint`）。

## 根因

叠加关系被混进了"选择模型"这一步：

- `applyCore` 的逻辑是**只取一层**：`joint > npoints > internal`；
- `with` 只是 `apply()` 阶段叠加的 offset，本不该影响"取哪一层"；
- 但 `with_source_` 为空/不为空的分支提前 `return`，把 `with` 当成了核心层 → 核心层为空 → `applyCore` 失败。

## 影响

- 用到随动补偿的场景全部受影响，且**看起来像是"标定数据错了"**，排查方向容易被带偏。

## 修法 / 教训

- 使用前先确认：配置 `with_source_` 时核心层是否仍然被正确设置。
- **API 的隐藏前置条件**（"配了 A 就别指望 B"）必须写进调用处注释，否则下一任维护者必然踩。
- 引出通用教训：**"叠加"和"选择"是两种语义**，不要用同一套分支逻辑同时处理。

## 关联

- [[projects/bordercollie/decisions/0004-calib-layer-priority]] —— 五层与叠加顺序
