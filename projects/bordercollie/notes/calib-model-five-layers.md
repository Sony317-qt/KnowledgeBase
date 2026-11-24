---
id: projects/bordercollie/notes/calib-model-five-layers
title: CalibModel 五层模型与 applyCore / apply 的区别
level: 2
tags: [calibration, bordercollie, geometry]
aliases: [CalibModel, applyCore, 9 点标定, 联合标定, 随动标定]
reviewed: 2026-11-20
next_review: 2026-11-22
projects: [bordercollie]
sources: [notes/bordercollie-study-notes.md 第 4 条主线, Modules/Industry/industryBase.h:20-165]
---

## 一句话

`applyCore` **只取一层**（`joint > npoints > internal`），`apply` 再在结果上依次叠加 `with`、`teach`——
"选择模型"和"叠加偏移"是两件事，混在一起就会出错。

## 机制

五层：

| 层 | 作用 | 生效顺序 | 实现 |
|---|---|---|---|
| `internal` | 畸变矫正 + 图像→世界（FFD 内参） | 最低 | `calibinternalthread.cpp:57-71` |
| `npoints` | 多点映射 + 角度修正 + 旋转中心 | 中 | `calibnpointsthread.cpp:132-142` |
| `joint` | 源相机→目标相机 + `robot_offset` | 最高 | `calibjointthread.cpp:83-89` |
| `with` | 随动 offset（如机器人随动） | `apply()` 时叠加 | `CalibModel::with` |
| `teach` | 对位示教偏移 | `apply()` 时最后叠加 | `CalibModel::teach` |

```cpp
applyCore(x, y, r)   // industryBase.h:151 —— 只用一层：joint > npoints > internal
apply(x, y, r)       // industryBase.h:158 —— applyCore 后依次叠加 with、teach
```

9 点平移流程：

```text
画 ROI / 学模板（或加载 IB .task）
 → 机器人到位，下发坐标（CalibPos:CCD1,index,x,y,r）或手填
 → 触发拍照 → 模板匹配取中心
 → AddPoint(img, tar) × 9
 → Calibrate()
 → ImageToWorld 反投影，算 RMS 物理误差
```

3 点旋转：`LCalibRotation` + `predict()` 判断角度模型（`y=x+c` 或 `y=-x+c`），求旋转中心与角度补偿；
模式有 `TranslationOnly / LocalRotation / DoublePointRotation`。通信索引 1–9 是平移点，10–12 是旋转点（`setPos` 里 `index--`）。

## 追问链

1. **`applyCore` 的优先级？**
   `joint > npoints > internal`，**只取一层**，不是层层叠加。
2. **`with` 和 `teach` 在哪一步？**
   `applyCore` 之后依次叠加 `with`、`teach`，顺序在 `apply()` 里固定，不要调换。
3. **9 点标定的输入输出？**
   输入 9 组（图像中心点 `tar` + 机器人坐标 `CalibPos`），输出映射矩阵；用 `ImageToWorld` 反投影算 RMS 物理误差作为精度指标。
4. **联合标定用哪个矩阵？**
   统一用**终点**的标定矩阵。

## 项目证据 / 反例

- **踩过的坑**：配置了 `with_source_` 时 `getModel()` **只算 `with.offset` 就 return**，核心层为空 → `applyCore` 失败（见 [[projects/bordercollie/incidents/2026-11-20-calib-with-only-offset]]）。
- 标定拍照走 `calibSoftTrigger` 分支，**不进生产算法链**。
- 标定数据是"型号级"的，换型会重新加载；标定没做完就跑生产会得到错误物理坐标。
- 运行时：`calib_model_map`（`Modules/Industry/locator`）供算法 `apply`；持久化在 `calib.json`（关闭标定窗口时写）。
- **反例**："所有层都在 `applyCore` 里叠加" ✗ —— `applyCore` 只取一层。

## 关联

- [[projects/bordercollie/decisions/0004-calib-layer-priority]]
- [[projects/bordercollie/README]]
