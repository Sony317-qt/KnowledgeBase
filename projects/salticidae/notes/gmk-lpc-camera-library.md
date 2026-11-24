---
id: projects/salticidae/notes/gmk-lpc-camera-library
title: LPC 相机库（lpcManager）读书笔记
level: 2
tags: [camera, c-api, plugin, architecture, gmk]
aliases: [LPC, lpcManager, salticidae 相机库, ICameraObject]
reviewed: 2026-11-20
next_review: 2026-11-22
projects: [salticidae]
sources: [doc/GMK学习笔记-整编版.md 第 2–5 章]
---

## 一句话

LPC 相机库用 **"6 个导出函数的 C ABI 插件 + `CameraCtrl` 包装层"**，把 60+ 家相机 SDK 藏到同一个属性模型后面；
读它的入口不是某个驱动，而是 `CameraCtrl` 与属性抽象。

## 机制

- **四层结构**：应用层 → API 层（`lpcManager.dll` 导出 `L*` 系列 C 函数）→ 插件层（`plugins\lpcXXX.dll`，每品牌一个）→ 基础层（`Core.dll` + OpenCV + lpv/PCL + log4cplus）。
- **三个对象**：
  - `CameraManager` 单例（`map<SN, Ctrl>` / `map<组名, Group>` / 配置持久化）
  - `CameraCtrl` 每台相机一个（状态机 / 属性缓存 / 回调中转 / 拆图 / Bayer）
  - `ICameraObject` 8 个纯虚函数，60+ 驱动实现

  **`CameraCtrl` 不继承 `ICameraObject`，是持有关系**——这是最容易记错的一处。
- **插件 ABI 是运行时字符串约定**（`Core::LibraryHelper::Excecute` 按名字查），不是编译期接口；`extern "C"` 只为防名字修饰。
  扫描规则写死在框架里：文件名 `lpc` 开头 + `.dll` + 能取到函数。一个坏 dll 只记日志并跳过，不拖垮全体。
- **PUSH 与 PULL** 的差别是"循环所有权"：PULL 由插件自建 `std::thread` 轮询取图（Artray/JAI/Euresys），PUSH 由 SDK 内部线程回调（Basler/Hik/Automv/Dahua）。
- **统一属性模型** = 属性名 + 描述 + 可设置元素（选项/范围/字符串列表）+ 当前值；元素类型与当前值类型**编译期配对校验**，配错直接 `throw`。
- **框架补的公共属性**（所有相机都有，驱动不用实现）：`SplitByRow`、`BayerMode`。

## 追问链

1. **`CameraCtrl` 和 `ICameraObject` 什么关系？**
   持有，不是继承。`ICameraObject` 是驱动契约，`CameraCtrl` 是框架侧包装层，同步、属性缓存、拆图这些框架能力都在 Ctrl 里，驱动不重复实现。
2. **一个 `lpc*.dll` 至少要导出哪几个函数？为什么是运行时约定？**
   `GetCameraDeviceInfo` / `InitializeLib` / `TerminateLib` / `InitializeAccessCtrl` / `CameraNewInstance` / `EnumAvailableCameras`。
   框架按名字 `GetProcAddress` 查——好处是插件可以与框架独立演进，代价是拼错名字不会编译报错，只在运行时报"驱动加载失败"。
3. **`LAddCamera` 为什么可能很慢？**
   它内部会**临时开一次相机**（Open → `SynCameraPropertyToCache` → Close）把真实参数读进缓存，所以"添加"比想象中慢，且相机被占用时会失败。
4. **读一个新驱动，怎么 60 秒判断它是 PUSH 还是 PULL？**
   ① 搜 `std::thread`，有 = PULL；② 看 `IStartCamera` 里调什么；③ 末参是函数指针 = PUSH，无参 = PULL；④ 搜 `mPushImage` 定位"帧从哪来"。

## 项目证据 / 反例

- **反例**："`ImageEventHandler.h` 是所有驱动的通用回调基类" ✗ —— `CImageEventHandler` 是 **Basler 私有类**；
  Automv / 海康是 C 接口，只能传函数指针，写 Handler 类是多余的。同在 Handler 模式，CameraDaHeng 就写在主 `.h` 里——**拆不拆文件纯属个人风格**。
- **反例**："`CameraNewInstance` 之后就能列能力" ✗ —— 能力清单要 `IRegister` 之后才拿得到。
- **反例**："`Core` 只服务相机" ✗ —— 72 个工程共用（含电源、运动控制）；`LibraryHelper` 也**不**负责扫描文件夹，只负责加载调用，扫描在 `CameraLibraryHelper`。
- **工程取舍**：SDK 头文件暴露有两派——Basler 把 SDK 头放进 `.h`（无 SDK 时 IDE 全红），Artray 藏在 `.cpp`、成员用 `void*` + `AsDevice()` cast（**100% 可读**）。
  Artray 是最后接入的驱动，等于总结了前 60 个的经验 → **新驱动建议走这一派**。
- **踩过的坑**：`CameraLibraryHelper::_AddDirectoriesToPath` 把 `plugins\` 下每个子目录追加进进程 `PATH`，**只增不减**，重复 `LInit` 会让 PATH 膨胀（F-07）。
- **踩过的坑**：回调持锁调用户代码 → 锁升级死锁（`CameraCtrlCallback.cpp:20`）：用户在图像回调里注销自己会去拿同一把锁。
  规避：回调里不做注册/注销/关闭相机，需要就投递到自己的队列异步处理。

## 关联

- [[cpp/concurrency/lock-scope-discipline]] —— 回调持锁死锁的正例
- [[projects/salticidae/decisions/0005-group-frame-by-bvalid]] —— 相机组的组帧判据
- [[projects/salticidae/README]]
