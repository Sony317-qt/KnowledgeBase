---
id: projects/bordercollie/decisions/0004-calib-layer-priority
status: 已采纳
date: 2026-11-20
---

# 0004. 标定五层：`joint > npoints > internal`，`with`/`teach` 最后叠加

## 背景

像素坐标要变成物理坐标，中间可能同时存在：镜头畸变矫正、多点平移映射、相机间联合标定、
机器人随动偏移、对位示教偏移。它们不能简单相加，各有生效顺序。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 一条公式全算 | 快 | 模型耦合，改一处崩全部 |
| B 多层独立模型 + 固定叠加顺序 | 每层可单独标定/替换 | 顺序成隐性契约，调换即出错 |
| C 每次现场手选组合 | 灵活 | 配置爆炸，易错 |

## 决定

选 B，五层固定：

| 层 | 作用 | 生效顺序 |
|---|---|---|
| `internal` | 畸变 + 图像→世界（FFD） | 最低 |
| `npoints` | 多点映射 + 角度修正 + 旋转中心 | 中 |
| `joint` | 源→目标 + `robot_offset` | 最高 |
| `with` | 随动 offset | `apply()` 时叠加 |
| `teach` | 示教 offset | `apply()` 时最后叠加 |

```cpp
applyCore(x, y, r)   // industryBase.h:151 —— 只用一层：joint > npoints > internal
apply(x, y, r)       // industryBase.h:158 —— applyCore 后依次叠加 with、teach
```

## 后果

- 放弃："把所有偏移都塞进一个矩阵"的简单性，换来**改标定层时不能随意调换顺序**的纪律。
- `with` 与 `teach` 的叠加顺序不同（`apply` 里固定），调换会得到有偏差的坐标。
- 联合标定统一用**终点**的标定矩阵。
- 标定数据是"型号级"的，换型会重新加载；标定没做完就跑生产会得到错误物理坐标。
- 已知坑：配置了 `with_source_` 时 `getModel()` **只算 `with.offset` 就 return**，核心层为空 → `applyCore` 失败（见 [[incidents/2026-11-20-calib-with-only-offset]]）。

## 可被追问

1. `applyCore` 的优先级是什么？——`joint > npoints > internal`，只取一层。
2. `with` 和 `teach` 在哪一步叠加？——`applyCore` 之后依次叠加 `with`、`teach`。
3. 9 点标定的输入输出？——输入 9 组（图像点 `tar`，机器人坐标 `CalibPos`），输出映射矩阵；用 `ImageToWorld` 反投影算 RMS 物理误差。
4. 为什么标定拍照走 `calibSoftTrigger` 分支？——不能进生产算法链，否则标定数据会被当生产结果。
