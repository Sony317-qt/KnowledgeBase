---
id: projects/bordercollie/incidents/2026-11-20-source-switch-old-frame
date: 2026-11-20
type: 时序陷阱（切换窗口内旧帧污染）
---

# 2026-11-20 · 切源期间触发生产，拿到的是旧参数拍的图

## 症状

刚用 `SwitchDataSource` 切到新数据源，紧接着拍照，算法结果和参数对不上（用旧源的曝光 / 增益 / ROI 拍的）。
偶发，且在手动调试时几乎复现不了。

## 定位

切源流程 `dialogCameraSalticidae.cpp:364`：

```text
SwitchDataSource(camera_id, data_src_id)
  ├─ 把目标源参数拷贝到当前相机并下发 SDK
  ├─ 期间断开 snapFinish，防旧帧进算法
  └─ 2D 线扫：额外重载配置文件
```

`snapFinish` 的断开/恢复是一个**窗口**。窗口内如果有软触发或 PLC 触发，就会拿旧参数拍图。

## 根因

- 切源是"把参数拷贝 + 下发 SDK"的**非原子过程**，中间存在参数与相机状态不一致的时刻。
- 2D 线扫还要额外重载配置文件，窗口更长。

## 修法 / 教训

- **切源期间不要触发生产**（这是使用约束，目前靠纪律而非机制保证）。
- 更稳的做法：切源期间把生产触发置为"忙"，或记录参数版本号随帧下发，让算法能识别"帧参数与期望不符"。
  同理 3D 数据源只有在桌面 OpenGL 后端下才创建 `Widget3d`（`widgetUnit.cpp:295`），也是"配置看起来对、但视觉不生效"的一类问题。
- 易错点汇总：
  1. 3D 数据源依赖 `AA_UseDesktopOpenGL`；
  2. 数据源参数是值语义，不能跨相机引用；
  3. 切源窗口内不要触发。

## 关联

- [[projects/bordercollie/decisions/0002-datasource-as-preset]]
