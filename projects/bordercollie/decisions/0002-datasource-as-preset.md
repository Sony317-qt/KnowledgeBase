---
id: projects/bordercollie/decisions/0002-datasource-as-preset
status: 已采纳
date: 2026-11-20
---

# 0002. 数据源是"一台相机的一套预设方案"，不是硬件

## 背景

同一台相机在不同工位、不同产品、不同光源组合下需要完全不同的参数组合。
如果每次现场都手工调参数，换型就是灾难。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 参数存相机内（相机自带用户配置） | 换相机即带 | 不同厂商支持不一，算法目录 / 光源绑定无法一起走 |
| B 全局参数表 | 集中管理 | 无法表达"同一相机多种场景"；无法按源独占目录 |
| C 数据源 = `(相机, 场景)` 的一套预设 | 与业务单元一一对应 | 参数是**值语义**，不能跨相机引用 |

## 决定

选 C。定义：**一台相机在某种拍摄条件下的一套预设方案**（`class DataSource`，`dialogCameraDefine.h:144`），
不是硬件、不是一帧图、不是运行时状态。

| 项 | 结论 |
|---|---|
| 内容 | 名字、软触发延时、2D 参数、2D 线扫参数、3D 参数、基恩士 3D 参数、虚拟相机路径 |
| 标识 | `(相机名, 源名)`；内部 `data_src_id` **0 基**，通信地址 `CCDx-y` **1 基** |
| 独占 | 算法目录 `model/<型号>/<相机>/<源>/`、管控、保存路径、光源绑定、绑码、通信地址、总览格子 |
| 业务单元 | 每个数据源对应一个 `WidgetUnit`（`WidgetUnit = 相机 + 数据源`） |
| 约束 | 不能跨相机共享；随动补偿、图像融合都限同一相机 |

切源流程：

```text
PLC / TCP 命令 SwitchDataSource
  → DialogCameraSalticidae::onSignalSwitchDataSource      dialogCameraSalticidae.cpp:470
  → switchDataSource(camera_id, data_src_id)              :364
       ├─ 把目标源参数拷贝到当前相机并下发 SDK
       ├─ 期间断开 snapFinish，防旧帧进算法
       └─ 2D 线扫：额外重载配置文件
  → 软触发拍照（onSignalSoftTrigger，:530）
```

持久化：`cameraSaveJson`（关闭相机窗口时，**重命名数据源是例外：立即保存**）、`cameraLoadJson`（换型 / 启动加载型号）。

## 后果

- 放弃："参数跟随硬件"的直觉——换相机不等于换配置，必须按 `(相机, 源)` 重新选。
- 参数是**值语义**（`CameraPro::data_src` 是 `vector<DataSource>`），不要指望跨相机引用。
- 风险：改完参数没关窗口就崩溃 → 丢配置；目录同步会删除"不在 srcList 里的目录"。
- 3D 数据源只有在**桌面 OpenGL 后端**下才会创建 `Widget3d`（`widgetUnit.cpp:295` 的 `AA_UseDesktopOpenGL` 判断）。

## 可被追问

1. 数据源是什么？不是什么？——是预设方案；不是硬件 / 不是一帧图 / 不是运行时状态。
2. `data_src_id` 是 0 基还是 1 基？——内部 0 基，通信地址 `CCDx-y` 1 基，两者差一。
3. 切源时为什么要断开 `snapFinish`？——防止切换期间到达的旧帧（旧参数拍的）进算法。
4. 配置什么时候落盘？——关窗口时；**重命名是唯一立即保存的例外**。
