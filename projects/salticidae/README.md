---
id: projects/salticidae
title: Salticidae —— LPC 相机库 + GMK 板端采图系统
role: 待补（维护 / 二次开发 / 学习走读）
period: 2026
stack: [C++, C API, Qt 5.9.4, Python(IB), WPF/.NET, CMake, MSBuild, TCP, Modbus]
status: 学习中（只读走读，未参与开发）
code_base: salticidae @ GMK_tzm, git describe = 1.4.29-50-ga8b4e698, 1,600 commits
sources: [doc/GMK学习笔记-整编版.md]
---

## 一句话

GMK 是**双目 + Z 轴扫描 + 景深融合**的采图系统：板端（GMKdemo.exe）驱动电机逐层拍摄，
把一组不同焦面的图交给 IntelliBlink 引擎做焦点堆叠，输出一张全焦图，再经 TCP 发给上位机。
它的全部复杂度来自同时踩在**四条边界**上：

```text
相机  （进程内 C 调用 + 回调）
PLC   （TCP ASCII 文本，板端作客户端）
上位机（TCP V3 二进制，板端作服务端）
IB    （进程内 C++ + CPython）
```

## 30 秒陈述

> TODO(你补)：这是"我做过什么"还是"我读懂了什么"，开场句必须分清——
> 走读型项目不能包装成主导设计，否则第一个追问就穿帮。

## 背景与约束

- **业务目标**：对被测件做景深融合——Z 轴逐层步进拍摄，每层一张，堆叠成全焦图（现场以光模块类样品为主）；双相机则左右各一路，各自融合。
- **硬约束**：帧间隔 ~8 ms、单帧 98.8 MiB（11264×9200 MONO8）→ 内存带宽是瓶颈，不是 CPU。
- **板端为什么是 Windows**：二期从 Linux/RK3588 迁到 Windows，直接复用 LPC 相机库与大恒 SDK；代价是实时性不如 RTOS，所以**运动节奏全部交给 PLC 内部执行**，板端只响应事件。

## 技术栈与环境

| 项 | 内容 |
|---|---|
| 语言 | C++（核心）、Python（IB 算法）、C# / WPF（上位机客户端插件）、C（相机 SDK） |
| 框架/库 | Qt 5.9.4 msvc2017_64、OpenCV（`lpOpenCV` 私有 fork）、IntelliBlink 3.8.86、lpv SDK 2.21.1、log4cplus、libmodbus、HslCommunication(.NET) |
| 构建 | MSBuild + Visual Studio 2017（v141）；Windows SDK 10.0.16299（工程写死） |
| 平台 | Windows 工控机（板端）；上位机客户电脑 |
| 硬件 | 大恒相机（GalaxySDK）、PLC（EL 单轴 / MMC 双轴，192.168.10.200:6000/6001） |

## 架构

```text
上位机客户端（lpcGMKWin 插件）
    │  TCP 9500 控制 / 9501 左 / 9502 右 ── 板端作【服务端】
    ▼
┌──────────────────────────────────────────────┐
│                GMKdemo（板端）                 │
│  GmkServer → 收指令、回传图像                  │
│  GMKBoard  → 运动 / 融合 / 调度（上帝类 3,426 行）│
│  MotorPlc  → 控制电机运动（TCP ASCII 客户端）   │
│  IBProcessingPipeline → 拆图 4 块 → 4 路融合   │
└──────────────────────────────────────────────┘
    │  TCP 6000 / 6001 ── 板端作【客户端】
    ▼
PLC 运动控制器（EL 单轴 / MMC 双轴）
```

**一帧的旅程（5 类线程）**：

```text
PLC 推 "3" → TriggerFrame → LSnapCamera → 相机驱动回调（驱动线程）
  → OnLpcFrame → EnqueueOriginalsRaw（从池取复用缓冲，1 次 memcpy）
  → ScheduleFrame → fusionQueues_[1]/[2]（左右独立）
  → FusionLoop(1)/(2) → RunFrameJob
  → IBProcessingPipeline::Process（拆图 → 4 路并行融合 → 合成）
  → PublishOutputs（TCP / UI / 存图）
  → MaybeFinishChannel → MaybeFinishTask
  → WaitChannelDrained → CHANNEL_DONE + TASK_DONE
```

跨 5 类线程：`ControlLoop` → 抓拍线程（detach）→ 相机回调线程 → `FusionLoop` → `SenderLoop`。

## 仓库形态：一个容器，两个仓库

```text
D:\git_IBSG\                    ← 容器目录（本身不是 git 仓库）
├─ salticidae\      独立仓库：相机库 + GMKdemo + 驱动插件（本项目主体）
├─ 3rdparty\        独立仓库：只存 SDK 清单壳（实体需内网获取）
├─ lpOpenCV\        私有 OpenCV fork（74 个工程写死相对路径）
├─ _qtvstools\      从 vsix 提取的 QtMSBuild
└─ Output\          构建产物落点
```

规模：`src` + `include` 约 370 文件 / 8.9 万行，其中 **60+ 相机驱动插件约占 76%**；框架 + 测试 + 工具约 1.7 万行。
GMKdemo 自身 35 文件 / 11,698 行。

## 关键决策

- [[decisions/0001-vision-not-plc-controlled]] —— 相机控制权在板端，PLC 只提供节拍
- [[decisions/0002-two-phase-async-capture]] —— 先同步 Prepare 再异步 StartCapture
- [[decisions/0003-five-transport-modes]] —— 一条 PLC 链路要五个收发入口，加锁解决不了
- [[decisions/0004-buffer-pool-and-delayed-reclaim]] —— 98.8 MiB/帧、8 ms 间隔，必须复用缓冲池
- [[decisions/0005-group-frame-by-bvalid]] —— 多相机"齐了"靠 `bValid` 判据，不靠时间戳

## 事故 / 认知修正

- [[incidents/2026-11-20-async-left-right-count]] —— 文档说左右独立张数，代码已强制共用
- [[incidents/2026-11-20-task-done-overtaken]] —— 大图比 `TASK_DONE` 晚到 → 客户端统计 0 张
- [[incidents/2026-11-20-plc-message-syntax]] —— 请求不带尖括号、两种分隔符
- [[incidents/2026-11-20-ipc-missing-blocks-main-chain]] —— IB 包缺失，主链路全部被拒

## 我的职责与产出

- **走读 / 学习**：对 `GMK_tzm` 分支做系统性代码走读，产出整编笔记（13 章 + 4 附录）。
- 梳理出的认知修正 20 条、质量问题台账 40+ 条（E/F/D/B/S/P/T/Q 八类编号）。

> 这一节决定了 level 上限：不是自己设计的项目，技术点**最高标 2**（见 README 的 level 规则）。

## 量化

| 项 | 数值 |
|---|---|
| 单帧大小 | 11264×9200 MONO8 ≈ 98.8 MiB |
| 帧间隔 | ~8 ms |
| 连续抓拍 | 一轮 14 次步进（`INTERVAL 1 14 0.04`） |
| 缓冲分配 | 一次 20–24 ms vs 帧间隔 8 ms → 必须复用 |
| GMKdemo | 35 文件 / 11,698 行 |
| 代码库 | ~370 文件 / 8.9 万行 / 1,600 commits（2023-05 起） |
| 主题 | IB / Niche / Other |
| 仓库路径 | `D:\git_IBSG\salticidae` |

## 用到的技术点

- [[cpp/memory/zero-copy]] —— `cv::Mat` 移动只偷指针；`LFrame` 的 `pData` 被驱动复用
- [[cpp/concurrency/lock-scope-discipline]] —— 锁内只做数据结构操作，I/O 与分配放锁外
- [[cpp/concurrency/condition-variable]] —— `inFlight` + `notify_all` 实现"排空"等待
- [[cpp/object-model/rule-of-five]] —— `PDUMotionWrapper` 析构为空 → 泄漏
- [[cpp/memory/noexcept-and-move]] —— `std::move` 入队是 O(1) 交棒

## 原始笔记（全文入库）

| 文件 | 大小 | 来源 |
|---|---|---|
| [[projects/salticidae/source/GMK学习笔记-整编版]] | 83,151 B / 1,796 行 | `D:\git_IBSG\salticidae\doc\GMK学习笔记-整编版.md` |

> 原文**未做任何改写**，逐字节复制（SHA256 `E4F32E45…D39A`）。
> 目录下其他文件是从它抽取的索引/ADR/事故/技术点；**原文里讲的完整机制、命令表、线程图、行号、速查表、自测 25 题，都以原文为准。**

## 面包屑

- [[meta/index]] · 在原文中快速定位：`rg '^# ' projects/salticidae/source/` 看编/章标题
