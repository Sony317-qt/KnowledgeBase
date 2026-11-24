---
id: projects/bordercollie/notes/datasource-as-preset
title: 数据源（DataSource）的语义边界
level: 2
tags: [camera, bordercollie, domain-model]
aliases: [数据源, DataSource, WidgetUnit, CCDx-y]
reviewed: 2026-11-20
next_review: 2026-11-22
projects: [bordercollie]
sources: [notes/bordercollie-study-notes.md 第 2 条主线]
---

## 一句话

数据源是**一台相机在某种拍摄条件下的一整套预设方案**——它独占算法目录、光源绑定、保存路径与通信地址，
是业务上的"一个检测工位"，而不是硬件、不是一帧图、也不是运行时状态。

## 机制

- 定义在 `class DataSource`（`dialogCameraDefine.h:144`）。
- 内容：名字、软触发延时、2D 参数、2D 线扫参数、3D 参数、基恩士 3D 参数、虚拟相机路径。
- 标识：`(相机名, 源名)`；内部 `data_src_id` **0 基**，通信地址 `CCDx-y` **1 基**。
- 独占：算法目录 `model/<型号>/<相机>/<源>/`、管控、保存路径、光源绑定、绑码、通信地址、总览格子。
- **业务单元**：每个数据源对应一个 `WidgetUnit`，定义是 `WidgetUnit = 相机 + 数据源`。
- 值是拷贝语义：`CameraPro::data_src` 是 `vector<DataSource>`。

## 追问链

1. **数据源是什么？不是什么？**
   是一套拍摄预设 + 业务单元；不是硬件、不是一帧图、不是运行时状态。
2. **为什么不能跨相机共享？**
   它绑定了该相机自己的参数体系与算法目录；换个相机，参数含义（传感器尺寸、ROI 可置范围）都变了。
   所以随动补偿、图像融合也都被限制在同一相机内。
3. **0 基还是 1 基？**
   内部 ID 0 基，对外通信地址 1 基，差一处就错件。
4. **什么时候落盘？**
   关闭相机窗口时（`cameraSaveJson`）；**重命名是唯一立即保存的例外**。改完没关窗口就崩溃 → 丢配置。

## 项目证据 / 反例

- 工业场景的通用需求：同一台相机 × 不同产品 × 不同光源，就是一个二维矩阵——数据源就是这个矩阵的格子。
- **踩过的坑**：切换数据源时若不断开 `snapFinish`，窗口内到达的旧帧会用旧参数进算法（见 [[projects/bordercollie/incidents/2026-11-20-source-switch-old-frame]]）。
- **踩过的坑**：3D 数据源只有在桌面 OpenGL 后端下才创建 `Widget3d`（`widgetUnit.cpp:295` 判断 `AA_UseDesktopOpenGL`）。
- **反例**："数据源可以跨相机引用" ✗ —— 值语义，拷贝之后各是各的。

## 关联

- [[projects/bordercollie/decisions/0002-datasource-as-preset]]
- [[projects/bordercollie/README]]
