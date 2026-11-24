---
id: projects/bordercollie/decisions/0001-single-ib-instance
status: 已采纳
date: 2026-11-20
---

# 0001. IB 服务全局只建一个实例，靠注入分发

## 背景

- `SetUp()` 要加载整个 IB 工程（任务 + 全局变量 + 算子库），开销大。
- IB 全局变量是**进程级共享**，多实例会互相冲突（代码里有 `globalVarConflict` 处理）。
- 但算法工厂、信号站、标定、内置定位器都需要 IB。

## 候选方案

| 方案 | 优点 | 代价 |
|---|---|---|
| A 谁用谁建 | 无共享状态 | 重复 `SetUp` 开销大；全局变量冲突 |
| B 单例（静态） | 全局唯一 | 生命周期不受控，卸载顺序难保证 |
| C 主窗口成员持有 + `ILIBServicePtr` 引用计数分发 | 唯一实例 + 生命周期归主程序 | 注入时机有硬约束 |

## 决定

选 C，加载时序固定在主窗口装配阶段（**不是 `main.cpp`**）：

```text
MainWindow 构造
  ├─ ib_service_ = new IBservice()                 mainwindow.cpp:186
  │    ├─ LIBService::Create()
  │    └─ 读 file/ib.json → "ib_path"
  │    └─ loadIB(path) → SetUp(path)
  │                    → LoadGlobalAction(task/globalAction.json)
  ├─ industry_->setIB(ib_service_->getIB())        mainwindow.cpp:193
  ├─ signal_station->setIB(ib_service_->getIB())   mainwindow.cpp:194
  ├─ calib_->setIB(ib_service_->getIB())           mainwindow.cpp:374
  └─ 切换行业 DLL 后重新 setIB                     mainwindow.cpp:2693
```

消费方：

| 消费方 | 用途 |
|---|---|
| `industry_`（DLL 工厂） | 只存不用的"中转站"，`setIB/getIB` 原样返回 |
| `WidgetAlg::locator_`（内置定位器） | 从 `industry_` 取回 IB，传给 `AlgIB` |
| `AlgIB` | 构造时 `LoadTask`；运行时 `ExecuteTaskSync`；卸载 `UnloadTask` |
| `SignalStation` | 通信参数读写 IB 全局变量 |
| `DialogCalib` | 多点标定里跑 IB 任务 |

## 后果

- 放弃："模块自治"——所有需要 IB 的模块现在都对主窗口有隐式依赖，必须**注入早于算子创建**（`AlgIB` 构造即 `LoadTask`）。
- `ib_path` 为空 / `SetUp` 失败 → `IsReady()==false` → `AlgIB` 创建失败 → 界面上加不出 IB 算子。
- `task/globalAction.json` 加载失败**只记 `qCritical`，不拦截启动**——和 salticidae 的 IB 缺失是同一类问题：初始化失败被降级成日志。
- 线程：IB 调用发生在算法线程（`widgetAlgThread`），不在 GUI 线程。

## 可被追问

1. IB 是谁创建的？——主窗口装配阶段，不是 `main.cpp`；`ib_path` 为空则不加载。
2. 为什么 `IndustryBase` 要提供 `setIB/getIB`，而 DLL 自己又不用？——它只是注入通道，真正消费者是 `AlgIB` 等。
3. 为什么只建一个实例？——`SetUp` 开销大 + 全局变量进程级共享。
