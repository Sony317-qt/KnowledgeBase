# GMK 系统与 LPC 相机库 学习笔记（整编版）

> **本书来源**：由《GMK 通信链路学习笔记》原稿（19 章 / 951 行）与《GMKWin.md》工程说明整编而成。
> **整编日期**：2026-11-20
> **代码基线**：`salticidae` @ 分支 `GMK_tzm`，`git describe = 1.4.29-50-ga8b4e698`，1,600 次提交。
> **阅读对象**：需要维护 / 二次开发 GMK 板端程序、接入 LPC 相机库的工程师。

---

## 整编说明（本版做了什么）

原稿是"会话流水"：分析一段、追加一段，所以同一件事常被讲三遍，问题清单攒了四份，认知修正攒了两份。本版做了五件事：

| # | 处理 | 说明 |
|---|---|---|
| 1 | **按书重排** | 起步（构建）→ 框架（相机库）→ 业务（GMK）→ 深化（并发/质量）→ 附录 |
| 2 | **合并 8 组重复** | PLC 链路、上位机链路、并发模型、速查表、问题清单、构建问题、认知修正、架构职责，详见附录 C |
| 3 | **四份问题清单并一份** | 按"环境 / 框架 / 驱动 / 业务 / 协议 / 构建 / 文档"归类，去掉重复项，统一编号 |
| 4 | **两份认知修正并一份** | 20 条按主题分组，不再按"第几轮"排列 |
| 5 | **核对数字** | 行数、提交数、目录是否存在等，均按当前代码复核；与原稿不符处标 **⚑ 核对** |

原稿文件保持不动，本版是**新增**文件。若两处说法冲突，以本书为准。

### 术语统一

原稿里"软件端 / 上位机 / 客户端"、"板端 / 下位机 / GMKdemo"混用，本版统一如下：

| 术语 | 指什么 | 代码位置 |
|---|---|---|
| **上位机** | 运行在客户电脑上的采图软件；GMK 场景下即 lpcGMKWin 插件所在进程 | `src/CameraGMKWin` |
| **板端** | 运行 GMKdemo.exe 的 Windows 工控机，同时是上位机的服务端、PLC 的客户端 | `src/GMKdemo` |
| **PLC** | 运动控制器（EL 单轴 / MMC 双轴），只管电机与光源 | 外部设备 |
| **相机** | 由板端通过 LPC 相机库直接控制，**不经过 PLC** | `src/CameraXxx` 插件 |
| **驱动 / 插件** | 一个 `lpc*.dll`，把某品牌相机适配成 `ICameraObject` | `src/CameraXxx` |
| **框架** | `lpcManager.dll` + `Core.dll`，负责加载、状态机、属性、配置、事件 | `src/CameraManager`、`src/Core` |

---

## 目录

**第一编 起步：这台机器怎么跑起来**

- 第 1 章 项目全貌
  - 1.1 一句话说清 GMK 是什么
  - 1.2 四个角色与两条通信链路
  - 1.3 仓库形态：一个容器，两个仓库
  - 1.4 两代客户端：CameraGMK 与 CameraGMKWin
- 第 2 章 构建与运行环境
  - 2.1 依赖全景
  - 2.2 构建顺序与产物落点
  - 2.3 三个"位置"决定成败
  - 2.4 从全红到能跑：九类报错对照
  - 2.5 用户级 MSBuild props 方案
  - 2.6 启动验证清单
  - 2.7 Windows DLL 搜索规则
  - 2.8 部署目录清单

**第二编 框架：LPC 相机库长什么样**

- 第 3 章 相机库分层与插件机制
  - 3.1 四层结构
  - 3.2 三个对象：Manager / Ctrl / ICameraObject
  - 3.3 装配链：从 LInit 到一台可用相机
  - 3.4 插件 ABI：六个导出函数
  - 3.5 两种取图模型：PUSH 与 PULL
  - 3.6 SDK 头文件暴露策略的两派
  - 3.7 `src/` 下四种性质并存
- 第 4 章 核心功能：对外能力清单
  - 4.1 API 全貌
  - 4.2 生命周期与状态机
  - 4.3 统一属性模型
  - 4.4 取图回调与 LFrame
  - 4.5 事件通知
  - 4.6 配置持久化
  - 4.7 错误码与日志
  - 4.8 虚拟相机：唯一的无硬件路径
- 第 5 章 相机组、同步与拆图
  - 5.1 组是什么
  - 5.2 组帧的判据：bValid，不是时间戳
  - 5.3 触发模式与同步保证
  - 5.4 组的六个属性
  - 5.5 拆图与拼图
  - 5.6 多相机组 vs 景深融合
  - 5.7 GMK IS 相机的特殊性
  - 5.8 已知缺陷

**第三编 业务：GMK 系统怎么工作**

- 第 6 章 GMK 系统总览
  - 6.1 业务目标
  - 6.2 部署形态
  - 6.3 四条外部边界
  - 6.4 配置文件 GMKdemo.ini
  - 6.5 启动流程
  - 6.6 抓拍模式
- 第 7 章 板端 GMKdemo 内部结构
  - 7.1 目录职责
  - 7.2 依赖方向
  - 7.3 核心数据结构
  - 7.4 一帧的旅程
  - 7.5 输入链路的七步优化
  - 7.6 任务状态机
  - 7.7 IB 融合的四层封装
  - 7.8 缓冲池与延迟回收
- 第 8 章 PLC 通信链路（对下）
  - 8.1 角色与端点
  - 8.2 报文格式
  - 8.3 命令表
  - 8.4 状态流与"循环在谁手里"
  - 8.5 懒连接与断链判定
  - 8.6 带超时的非阻塞 connect
  - 8.7 五种收发模式
  - 8.8 并发保护
  - 8.9 线程模型
  - 8.10 EL 与 MMC
  - 8.11 一次连续抓拍的时序全景
- 第 9 章 上位机通信链路（对上）
  - 9.1 三端口
  - 9.2 协议来源与项目改动
  - 9.3 帧格式
  - 9.4 六条返回链路
  - 9.5 线程模型
  - 9.6 发送队列与背压
  - 9.7 跨口超车与 WaitChannelDrained
  - 9.8 性能调优
  - 9.9 上位机客户端插件
  - 9.10 会话与安全限制

**第四编 深化：并发、质量与结论**

- 第 10 章 并发与线程模型
  - 10.1 两套并发问题
  - 10.2 优先级规则
  - 10.3 拒绝矩阵
  - 10.4 三层异步防线
  - 10.5 锁顺序与 RAII
  - 10.6 锁外操作原则
  - 10.7 取消机制
  - 10.8 内存与拷贝
  - 10.9 一个死锁案例
- 第 11 章 质量与风险台账
  - 11.1 环境与部署
  - 11.2 框架层
  - 11.3 驱动层
  - 11.4 板端业务
  - 11.5 协议与服务器
  - 11.6 PLC 封装（PDUMotionWrapper）
  - 11.7 构建与工程
  - 11.8 文档与测试
  - 11.9 本会话已处理项
- 第 12 章 认知修正录
  - 12.1 分层与职责
  - 12.2 通信与协议
  - 12.3 并发与时序
  - 12.4 相机到底受谁控制
- 第 13 章 结论与下一步
  - 13.1 五条结论
  - 13.2 待确认事项
  - 13.3 下一步

**附录**

- 附录 A 概念速查（网络 / 相机库 / GMK 业务 / 板间通信 / 构建）
- 附录 B 配置与部署速查
- 附录 C 重叠审计与原稿对照表
- 附录 D 复习自测 25 题

---

# 第一编 起步：这台机器怎么跑起来

# 第 1 章 项目全貌

## 1.1 一句话说清 GMK 是什么

GMK 是**双目 + Z 轴扫描 + 景深融合**的采图系统：板端驱动电机逐层拍摄，把一组不同焦面的图交给 IntelliBlink 引擎做焦点堆叠，输出一张全焦图，再通过 TCP 发给上位机。

它同时踩在四条边界上，这也是全部复杂度的来源：

```
相机（进程内 C 调用 + 回调）
PLC （TCP ASCII 文本，客户端）
上位机（TCP V3 二进制，服务端）
IB 引擎（进程内 C++ 调用 + CPython）
```

## 1.2 四个角色与两条通信链路

```
上位机客户端（CameraGMKWin 插件）
    │  TCP 9500 / 9501 / 9502  ── 板端作【服务端】
    ▼
┌──────────────────────────────────────┐
│            GMKdemo（板端）             │
│  GmkServer   ← 收指令、回传图像        │
│  GMKBoard    ← 运动 / 融合 / 调度      │
│  MotorPlc    → 控制电机运动            │
└──────────────────────────────────────┘
    │  TCP 6000 / 6001  ── 板端作【客户端】
    ▼
PLC 运动控制器（EL 单轴 / MMC 双轴）
```

| 角色 | 位置 | 职责 | 代码 |
|---|---|---|---|
| 上位机 | 最上层 | 发指令、显示图像 | `src/CameraGMKWin`、`src/CameraGMK` |
| **板端** | 中间层 | 对上作服务端、对下作客户端，负责时序协调 | `src/GMKdemo` |
| PLC | 底层 | 执行电机运动、回报状态 | 外部设备（6000/6001） |
| 相机 | 底层 | 被板端直接配置与触发取图 | LPC 相机库插件 |

**最容易被误解的一点**：相机**不是** PLC 控制的。PLC 只走到位后推一个状态码 `3` 表示"该拍了"，板端收到 `3` 才去调 `LSnapCamera`。

```
PLC 走一步 ──推状态码 3──▶ 板端 onTrigger ──▶ 触发左相机 + 触发右相机
```

## 1.3 仓库形态：一个容器，两个仓库

```
D:\git_IBSG\                    ← 容器目录（本身不是 git 仓库）
├─ salticidae\      独立仓库：相机库 + GMKdemo + 驱动插件（本书主体）
├─ 3rdparty\        独立仓库：只存 SDK 清单壳（实体需内网获取）
├─ lpOpenCV\        私有 OpenCV fork（74 个工程写死相对路径）
├─ _qtvstools\      从 vsix 提取的 QtMSBuild
└─ Output\          构建产物落点
```

**规模（⚑ 按当前代码复核）**：

| 项 | 数值 |
|---|---|
| `src` + `include` | 约 370 文件 / 8.9 万行 |
| 相机驱动插件 | 60+ 个，约占 76% 代码量 |
| 框架 + 测试 + 工具 | 约 1.7 万行 |
| 提交历史 | 1,600 次（2023-05 起） |

> ⚑ 核对：原稿记"1,629 次提交、HEAD 1.5.6、61 个插件"。因当时在另一分支，当前是 `GMK_tzm` / `1.4.29-50`。数量级不变，细节以本书为准。

## 1.4 两代客户端：CameraGMK 与 CameraGMKWin

仓库里有两个上位机插件，**不是备份关系，是两代实现**：

| 插件 | 协议 | 说明 |
|---|---|---|
| `src/CameraGMKWin` | TCP V3（当前） | 与板端 `kMagic`/`kVersion` 完全一致；双数据通道 |
| `src/CameraGMK` | 旧版 / 另一路 | 早期单通道实现，保留兼容 |

对上位机软件来说，它们都只是"一台 GMK 相机"：打开、抓图、收图，底层其实是 TCP。区别在于枚举出的设备形如 `host|port|left`、`host|port|right`（双目各一个逻辑设备）。

> ⚑ 核对：原稿第 6 章描述的是这一层；第 9 章讲的多相机组是**相机库内部**的另一回事，两者不要混。

---

# 第 2 章 构建与运行环境

> 本章对应原稿的"构建环境搭建实录"「本章关键概念」两处，合并了散落在问题清单里的 5 条部署问题。

## 2.1 依赖全景

| 依赖 | 版本 / 位置 | 是否必须 |
|---|---|---|
| Visual Studio | 2017（v141）；VS2022 + v141 工具集亦可 | 必须 |
| Windows SDK | 10.0.16299（工程写死） | 必须，缺了报 MSB803 |
| Qt | 5.9.4 msvc2017_64（GMKdemo / QtDemo 用） | 编译板端必须 |
| OpenCV | `lpOpenCV` 私有 fork | 必须 |
| GalaxySDK | `3rdparty\camera_DahengCXP\GalaxySDK` | 大恒相机必须 |
| lpv SDK | `3rdparty\lpv_sdk_2.21.1.0\bin\x64` | IB 融合必须 |
| IntelliBlink | `IB_3.8.86_x64_softEnc`（含 4 个 `.task`） | 融合主链路必须 |
| gtest | `salticidae\3rdparty\gtest`（含 NuGet 包） | 跑单测必须 |

## 2.2 构建顺序与产物落点

**顺序有依赖**，不能一把梭：

```
Core → CameraManager(lpcManager.dll) → 各 lpc* 驱动 → lpcGMKWin / lpcDaHeng → GMKdemo
```

产物统一落在 `$(BinaryOutDir)` 下：

```
Output\x64\Release\runner17\salticidae\
    GMKdemo.exe
    lpcManager.dll  Core.dll
    plugins\  (lpcHik.dll … lpcVirtual.dll …)
    GalaxySDK\  data\  GMKdemo.ini
```

**两个决定落点的变量**：

| 变量 | 影响 |
|---|---|
| `$(MSBuildThisFileDirectory)` | `salticidae.props` 放哪，`Output\` 就在哪 |
| `$(SolutionName)` | 走解决方案编译时有值（多一层 `salticidae\`）；命令行单编 `.vcxproj` 时为空（少一层） |

## 2.3 三个"位置"决定成败

这一节是整章的核心，三条都属"配置没错、就是找不到"的类型：

1. **`salticidae.props` 必须放在仓库父目录**。74 个工程写死 `..\..\..\salticidae.props`，而 props 里 `RootDir = $(MSBuildThisFileDirectory)Output\`：

   | props 位置 | `RootDir` 展开 |
   |---|---|
   | `D:\git_IBSG\salticidae.props` | `D:\git_IBSG\Output\` ✅ |
   | `D:\git_IBSG\salticidae\salticidae.props` | `D:\git_IBSG\salticidae\Output\` ❌ |

2. **`$(SolutionName)` 为空会少一层目录**（见上表）。

3. **DLL 必须和 exe 同级**，Windows 不搜父目录（见 2.7）。

> ⚑ 核对：当前 `D:\git_IBSG\salticidae.props` 与 `salticidae\salticidae.props` **两份都在**（后者是 untracked 副本）。保持根目录那份为准，仓库内那份只是为了单编工程时能命中。

## 2.4 从全红到能跑：九类报错对照

| # | 报错 | 根因 | 解法 |
|---|---|---|---|
| ① | `MSB803 找不到 Windows SDK 10.0.16299` | 该 SDK 未安装 | 安装 `winsdksetup_10.0.16299.exe` |
| ② | `LNK1181 无法打开 Core.lib` | props 不在仓库父目录 | 复制到 `D:\git_IBSG\salticidae.props` |
| ③ | `C2440 constexpr wchar_t*` | VS2026 严格模式（`/permissive-`） | 改 `constexpr const wchar_t*` |
| ④ | `C2664 cv::FileNode 参数不匹配` | 同上，临时值不能绑定非 const 引用 | 拆成 `auto tem = fs.root();` |
| ⑤ | `找不到 Qt.props` | Qt VS Tools 扩展未装 | 从 `.vsix` 提取 `QtMSBuild` 目录 |
| ⑥ | Qt VS Tools 装不上 VS2026 | manifest 限 `[17.12, 18.0)`，页面文案却写"支持 2026" | 绕过扩展，手动配 `QtInstall` |
| ⑦ | `qmake 找不到` | `QtInstall` 要的是版本名，需查注册表 | 改绝对路径 `C:\Qt\Qt5.9.4\5.9.4\msvc2017_64` |
| ⑧ | `LPV core/geom/IB.dll 缺失` | `GMKdemo.vcxproj` 的 `CopyLpvRuntime` 依赖 `$(GMKLPVBin)` 未定义 | 用户级 props 定义 `GMKLPVBin`/`GMKIBDir`/`GMKGalaxySdk` |
| ⑨ | `0xC0000135 DLL_NOT_FOUND` | `lpcManager.dll` 在子目录 | 复制到 exe 同级 |

## 2.5 用户级 MSBuild props 方案

路径：`%LOCALAPPDATA%\Microsoft\MSBuild\v4.0\Microsoft.Cpp.x64.user.props`

```xml
<PropertyGroup Label="UserMacros">
  <GMKLPVBin>D:\git_IBSG\3rdparty\lpv_sdk_2.21.1.0\bin\x64</GMKLPVBin>
  <GMKIBDir>D:\git_IBSG\3rdparty\_IntelliBlink_NOT_INSTALLED</GMKIBDir>
  <GMKGalaxySdk>D:\git_IBSG\3rdparty\camera_DahengCXP\GalaxySDK</GMKGalaxySdk>
</PropertyGroup>
```

**关键技巧**：`GMKIBDir` 故意指向**不存在的目录**，让 `Condition="Exists(...)"` 为假 → Copy 任务跳过 → 避免 `MSB3030 找不到 lpbengine.dll`。

**为什么不用改 vcxproj**：改工程文件会污染 git 工作区，且每个人机器路径不同；用户级 props 只在本机生效。

## 2.6 启动验证清单

跑通的标准输出（缺一项都说明环境不完整）：

```
[GalaxySDK] 使用 ...\salticidae\GalaxySDK          ✅
[LPC] LInit(...\salticidae) -> 0x0                 ✅
Licensed successfully via SENSE dongle             ✅
[ib] initialize failed: IB 目录不存在:
     E:\GMKTEST\IB_3.8.86_x64_softEnc              ❌ 业务组件缺失
GMKdemo exiting rc=0                                （主动退出，非崩溃）
```

最后一行 `rc=0` 是**主动退出**，不是崩溃——排查时别把它当异常。

## 2.7 Windows DLL 搜索规则

**Windows 只在 exe 所在目录（及 PATH）查找，不搜索父目录。**

```
runner17\                     ← 有 lpvCore.dll，但不会被找到
    └─ salticidae\GMKdemo.exe ← 只在这层找
```

这就是"外层明明有 DLL，内层 exe 却报 0xC0000135"的原因——**不是配置错误，是搜索规则**。

LPC 自己也踩这个坑：`CameraLibraryHelper::_AddDirectoriesToPath` 把 `plugins\` 下每个子目录追加进进程 `PATH`（厂商 DLL 依赖），代价是**只增不减**，重复 `LInit` 会让 PATH 膨胀。

## 2.8 部署目录清单

| 目录 / 文件 | 内容 | 谁部署 |
|---|---|---|
| `GMKdemo.exe` | 板端主程序 | 构建 |
| `lpcManager.dll`、`Core.dll` | 框架 | 构建 |
| `plugins\` | 各 `lpc*.dll` + 厂商运行库 | 构建 |
| `GalaxySDK\` | 大恒 APIDll / GenICam / GenTL | 构建（`CopyGalaxy`） |
| `data\` | LPC 日志配置 `sae.properties` | 构建 |
| `GMKdemo.ini` | 板端配置模板 | 构建复制 |
| lpv 运行库 + `lpbengine.dll` | 融合依赖 | `CopyLpvRuntime`（需 `$(GMKLPVBin)`） |
| IB 目录 | `IB_3.8.86_x64_softEnc` + 4 个 `.task` | **手工，且常缺失** |

**大恒相机枚举的前置条件**：`GxIAPI` 通过 `GENICAM_GENTL64_PATH` 找 GenTL（`.cti`）。GMKdemo 启动时按 `[Camera] galaxy_sdk` 自动设 `PATH` 与 `GENICAM_GENTL64_PATH`，**无需手工配环境变量**，只要 `GalaxySDK\` 随程序部署。

---

# 第二编 框架：LPC 相机库长什么样

# 第 3 章 相机库分层与插件机制

## 3.1 四层结构

```
应用层     C/C++ 程序 │ C# (lpc.Net.dll) │ QtDemo │ MFC Demo │ GMKdemo
                        │
API 层     lpcManager.dll  ← 导出 L* 系列 C 函数
           ├─ CameraManager   单例：注册表、插件加载、配置、事件、帧率统计
           ├─ CameraCtrl      每台相机一个：状态机、属性缓存、回调广播、拆图
           └─ CameraGroup     相机组：同步触发、超时、缓存队列、拼图
                        │
插件层     plugins\lpcXXX.dll  ← 每品牌一个，导出同一套 6 个函数
                        │
基础层     Core.dll（库加载/日志/线程池/Any/Timer）+ OpenCV + lpv/PCL + log4cplus
```

## 3.2 三个对象：Manager / Ctrl / ICameraObject

```
CameraManager（单例，全局：map<SN,Ctrl> / map<组名,Group> / 配置持久化）
      │ 持有
      ▼
CameraCtrl（1 台相机 1 个：状态机 / 属性缓存 / 回调中转 / 拆图 / Bayer）
      │ 持有（has-a，非继承！）
      ▼
ICameraObject（8 个纯虚函数，60+ 驱动实现）
```

**`CameraCtrl` 不继承 `ICameraObject`，是持有关系**。这是最容易记错的一处：`ICameraObject` 是驱动契约，`CameraCtrl` 是框架侧的包装层，同步、属性缓存、拆图这些框架能力都在 Ctrl 里，驱动不重复实现。

## 3.3 装配链：从 LInit 到一台可用相机

```
LInit 扫描 plugins\lpc*.dll
  → GetProcAddress 校验 GetCameraDeviceInfo + InitializeLib
  → 注册进 mCameraLib

LAddCamera(设备名, SN)
  → CameraNewInstance 造对象（拿到 shared_ptr<ICameraObject>）
  → make_shared<CameraCtrl>() 包一层
  → SetCameraObj → RegisterCamera
      ├ 构造 5 个回调绑定到 CameraCtrl 自身
      ├ 驱动 IRegister → 注册属性表 + 存回调
      └ 框架补 SplitByRow / BayerMode  ← 驱动不用重复实现
  → Open / SynCameraPropertyToCache / Close（临时开相机抓参数）
  → AddCameraCtrlObj 进全局 map
```

注意最后一步：`LAddCamera` 内部会**临时开一次相机**把真实参数读进缓存，所以"添加"比想象中慢，且相机被占用时会失败。

## 3.4 插件 ABI：六个导出函数

| 函数 | 作用 |
|---|---|
| `GetCameraDeviceInfo` | 报设备名与描述（框架据此建索引） |
| `InitializeLib` / `TerminateLib` | 全局初始化 / 反初始化厂商 SDK |
| `InitializeAccessCtrl` | 注入属性读写屏蔽表（插件旁同名 `.json`） |
| `CameraNewInstance` | 工厂：造一个 `ICameraObject` |
| `EnumAvailableCameras` | 枚举在线相机 SN |

**约定要点**：

- 六个函数是**运行时字符串约定**（`Core::LibraryHelper::Excecute` 按名字查），不是编译期接口；`extern "C"` 只为防名字修饰。
- 扫描规则写死在框架里：文件名以 `lpc` 开头、扩展名 `.dll`、能取到上述函数，才认。
- 一个 dirty dll 不会拖垮全体：加载失败只记日志并跳过。
- `CameraNewInstance` 只是建对象，**能力清单要 `IRegister` 之后才拿得到**。

## 3.5 两种取图模型：PUSH 与 PULL

| 模型 | 循环所有权 | 代表 | 特征 |
|---|---|---|---|
| **PULL 拉流** | 插件自建 `std::thread` | Artray、JAI、Euresys | `RetrieveNextBuffer` 轮询，断流/坏帧全自己判 |
| **PUSH 回调** | SDK 内部线程 | Basler、Hik、Automv、Dahua | 注册回调即可，`IStartCamera` 一行 |

**60 秒判断法**（读任意驱动先做这四步）：

```
① 搜 std::thread     → 有 = PULL
② 看 IStartCamera 里调什么
③ 末参是函数指针     → PUSH；无参 → PULL
④ 搜 mPushImage 定位"帧从哪来"
```

**`ImageEventHandler.h` 只属于 pylon 系**：

```cpp
class CTPImageEventPrinter : public CImageEventHandler   // ← 来自 pylon SDK
```

- `CImageEventHandler` 是 **Basler 私有类**，其他 SDK 没有；
- Automv / 海康是 C 接口，只能传函数指针，**写 Handler 类是多余的**；
- 同在 Handler 模式，`CameraDaHeng` 写在主 `.h` 里 → **拆不拆文件纯属个人风格**。

## 3.6 SDK 头文件暴露策略的两派

| | Basler | Artray |
|---|---|---|
| SDK 头位置 | `CameraBasler.h` | `CameraArtray.cpp` |
| 成员声明 | `CInstantCamera mCamera;` | `void* mpDevice;` |
| 调用前 cast | 不需要 | 需要（`AsDevice()`） |
| `.h` 依赖 | 必须能找到 pylon | 只依赖本项目 4 个头 |
| 可读性 | 无 SDK 时 IDE 全红 | **100% 可读** |

Artray 注释明说理由："为不让 SDK 头进 `.h` 污染上层"。它是最后接入的驱动，等于总结了前 60 个的经验——**新驱动建议走这一派**。

## 3.7 `src/` 下四种性质并存

命名无法区分，读目录时必须先分类：

| 类别 | 代表 |
|---|---|
| 框架层 | `CameraManager/`、`Core/` |
| 业务层 | `CameraGMK`（TCP 下位机）、`CameraExDLP`（标定）、`CameraExOCT`（重建） |
| 纯厂商标配 | 57 个 `CameraXxx` |
| 工具 | `CameraVirtual`（图片回放，唯一可无硬件跑） |
| 模板 | `CameraBulaBula`（`IStartCamera` 返回 `FunNotImplemented`） |

---

# 第 4 章 核心功能：对外能力清单

> 本章把相机库"能干什么"一次讲完，后面各章只讲"怎么实现"。接口定义见 [LCameraFunctions.h](../include/LCameraFunctions.h)。

## 4.1 API 全貌

| 组 | 主要函数 | 说明 |
|---|---|---|
| 初始化 | `LInit` / `LUnInit` | `LInit` 传插件目录，返回 `0` 为成功 |
| 设备 | `LEnumAvailableCameraDevice` / `LEnumAvailableNonTempCameraInfo` | 先枚举驱动，再按驱动枚举相机 |
| 相机 | `LAddCamera` / `LAddTempCamera` / `LDeleteCamera` / `LEnumCameraInfo` / `LGetCameraInfo` / `LGetCameraStatus` | 临时相机不写入配置文件 |
| 属性 | `LEnumCameraProperty` / `LGetCameraProperty` / `LGetCameraPropertyEx` / `LSetCameraProperty` | `Ex` 版带范围/选项/当前值 |
| 动作 | `LOpenCamera` / `LCloseCamera` / `LStartCamera` / `LStopCamera` / `LSnapCamera` | 顺序有硬约束 |
| 回调 | `LRegisterCameraImageCallBack` / `LUnregisterCameraImageCallBack` | 建议在 `Open` 前注册 |
| 相机组 | `LAddCameraGroup` / `LCameraGroupAddCamera` / `LOpenCameraGroup` / … | 与单相机一一对应 |
| 事件 | `LRegisterEventNotifyCallBack` / `LUnregisterEventNotifyCallBack` | 按位或组合订阅 |
| 配置 | `LSaveConfig` / `LLoadConfig` / `LCleanUp` | 传目录，不是文件名 |
| 辅助 | `LPopupMsg` / `LStr2WStr` / `LGetCurrentModulePath` / `LIsEqual` / `LIsEqualRange` | 错误码转文案等 |

## 4.2 生命周期与状态机

```
Init → 枚举驱动 → 枚举相机 → AddCamera → Register回调 → Open → Start → (Snap) → Stop → Close → UnInit
```

**硬约束**：

- 未 `Open` 不能 `Start`；
- 内触发模式下调 `Snap` 返回 `LCameraErrorCodeTriggerModeError`；
- 驱动可声明 `CAMERA_SET_OPERATE_NOT_NEED_OPEN()`（虚拟相机就是），此时才允许免开相机设参数。

**规模限制**（写在 [LCameraDefine.h](../include/LCameraDefine.h)）：

| 常量 | 值 |
|---|---|
| `L_CAMERA_MAX_DEVICE_NUM` | 65 类驱动 |
| `L_CAMERA_MAX_CAMERA_SIZE` | 100 台相机 |
| `L_CAMERA_MAX_CAMERA_GROUP_SIZE` | 100 个组 |
| `L_CAMERA_MAX_SINGLE_GROUP_SIZE` | 每组 10 台 |
| `L_CAMERA_MAX_PROPERTY_SZIE` | 每台 100 个属性 |

**状态位** `LDeviceStatus` 一个字节看全：IsInit / IsOccupy / IsOpen / IsStart / IsOnLine。

## 4.3 统一属性模型

不管哪家相机，参数都被抽象成：

```
属性名 + 描述 + 可设置元素（选项列表 / 数值范围 / 字符串列表）+ 当前值
```

**元素类型与当前值类型必须配对**（`PropertyType<>` 在编译期校验）：选项列表 → 字符串；范围 → 数值；字符串列表 → 字符串列表。配错会 `throw`。

**每个属性还带三条元信息**：

| 元信息 | 含义 |
|---|---|
| `bVolatile` | 易变属性，每次直读相机，不缓存、不同步 |
| `bNeedStopCamera` | 改这个参数前自动停相机 |
| `listNeedUpdateProperty` | 改完要联动刷新的属性（如 OffsetX 影响 Width） |

**属性读写屏蔽**：插件旁放同名 `.json`（`read_config_list` / `write_config_list`），就能在客户现场锁死某些参数。被屏蔽时 `LGetCameraProperty` 返回空，`Ex` 版返回 `DataTypeError`。

**框架补的公共属性**（所有相机都有，驱动不用实现）：

| 属性 | 作用 |
|---|---|
| `SplitByRow` | 按行把一帧拆成 1~8 张（分时频闪） |
| `BayerMode` | Bayer 转 RGB 的速度/精度档位（High Speed / High Precision） |

## 4.4 取图回调与 LFrame

`LRegisterCameraImageCallBack` 注册后，驱动线程回调 `LFrame`。`pNext` 把多张图串成链表——**一次回调可以带多张图**。

| 字段 | 用途 |
|---|---|
| `nFrameNum` | 帧号 |
| `lCamDevTimeStamp` | 相机曝光时间戳（无接口时为回调时间） |
| `lCamCallbackTimeStamp` | 回调触发时间戳 |
| `enMatType` | 普通 2D / EPG / 3D 点云 / Mesh / 点云+Mesh / 点云+2D |
| `pFrameData` | 图像数据（宽、高、像素格式、裸指针） |

两点要注意：**回调运行在驱动线程**，不要在里面做长耗时操作；驱动会复用 `pData`，需要留存必须自己拷。

## 4.5 事件通知

| 事件 | 携带数据 | 典型用途 |
|---|---|---|
| 相机掉线 / 上线 | `LStateEventData` | 界面提示、自动重连 |
| SDK 与驱动版本不匹配 | 同上 | 部署排障 |
| Open / Close / Start / Stop | 同上 | 状态同步 |
| 属性变化 | `LPropertyChangeEventData` | UI 刷新属性树 |
| 运行时信息 | `LPushInfoEventData` | 帧率 + 帧抖动（每 500ms） |
| 编码器脉冲 | `LPluseEventData` | 位置触发 |
| 通用 / 帧信号 | `LGeneralSignalEventData` | 外部计数 |
| DLP 标定结果 | `LDLPEventData` | 结构光标定流程 |

## 4.6 配置持久化

`LSaveConfig(目录)` 生成：

```
指定目录\
  salticidae\
    camera\ 1.json 2.json …
    group\  1.json 2.json …
```

保存内容：品牌、序列号、驱动版本、全部属性名/范围/当前值、组与相机的绑定关系。**临时相机与临时组不保存**。

> ⚑ 已知缺陷：保存前会 `remove_all` 再重建，目标目录里若有别的文件会被误删（见 F-05）。

## 4.7 错误码与日志

- **分界线 `0xFF`**：小于它是错误，大于等于它是**警告**（如 `BuffTooShort`、`配置未全量加载`）。警告不该按失败处理。
- `LPopupMsg(code)` 直接取中文文案，不用自己维护对照表。
- 日志用 log4cplus，默认 `${AppData}\salticidae\logs`；配置在运行目录 `data\salticidae\config\log4cplus.properties`。
- 驱动内部失败的细节只进日志，API 只返回 `CameraOpenFailed` 这类大类错误——**排障必须看日志**。

## 4.8 虚拟相机：唯一的无硬件路径

`lpcVirtual` 读磁盘图片序列或 3D 文件当相机用：

| 能力 | 说明 |
|---|---|
| 内存模式 | 预加载到内存（帧率要求高的场合）或只存路径 |
| 循环 | 播完从头再来 |
| 帧率 0 | 有多快读多快，但不丢图 |
| 拆图 / Bayer | 同样支持，用于验证公共属性 |

它是**唯一能无硬件跑通全链路**的驱动，也是 CI 与新人上手的入口。

---

# 第 5 章 相机组、同步与拆图

> 本章合并原稿 9.5（多相机同步）与 9.6（景深融合辨析）两节，并补上组的属性与已知缺陷。

## 5.1 组是什么

相机组（CameraGroup）把多台物理相机绑成一个逻辑设备，对外**只有一个组级回调**。内部：

```cpp
GroupFrame = unordered_map<std::wstring /*SN*/, CameraFrameInfo>
```

**键是 SN，不是时间戳**。这种设计的含义贯穿全章。

## 5.2 组帧的判据：bValid，不是时间戳

```
Snap → for 循环串行 SnapCamera() → 清帧槽
   → 各相机回调填自己槽位
   → 所有槽 bValid → 整组入队 → 推送线程 CherryPick 串成 LFrame 链表
   → 用户一次回调拿到 N 张（do { ... } while (pFrame = pFrame->pNext)）
```

**核心认知**：

- "齐了"的判据是"每个 SN 槽位 `bValid`"，**不是时间**；
- 没有 `frameId`，依赖"一次只有一批在飞"的隐含前提；
- 作者 TODO 已承认：**触发快于取图时，新旧图会被拼成一组**。

## 5.3 触发模式与同步保证

| 触发模式 | 同步保证 |
|---|---|
| `External`（外触发） | ✅ **真硬件同步**（外部 PLC 扇出） |
| `Continues`（组内触发） | ❌ Windows 定时器 + 串行下发 |
| `Single Frame`（软触发） | ❌ 用户调用 + 串行下发 |

**结论**：要精度只能走 `External`。组内触发只是"看起来同时"，实际是串行发命令。

## 5.4 组的六个属性

| 属性 | 作用 |
|---|---|
| `SynTriggerTime` | 同步触发时间（组内触发模式下有效） |
| `SynTimeOut` | 等帧超时；超时后推已有残帧 |
| `CacheQuantity` | 缓存队列深度（默认 10） |
| `ReceiveImage` | 使能收图（关掉可跳过整流而不停相机） |
| `SplitByRowIndex` | 拆图后取第几块（配合 `SplitByRow` 分流） |
| `AppendLastRows` | 拼图：把多台相机的行拼成一条长图 |

## 5.5 拆图与拼图

这是两个方向相反、常被混淆的功能：

| | SplitByRow（拆） | AppendLastRows（拼） |
|---|---|---|
| 层级 | 单相机 / 组都会继承 | 仅组 |
| 输入 | 一帧高图 | 多台相机各一段 |
| 输出 | 多帧矮图（`pNext` 串起来） | 一条长图 |
| 场景 | 分时频闪、多光源分时曝光 | 多相机拼接覆盖 |

## 5.6 多相机组 vs 景深融合

**这是全笔记最容易混的一处**：

| | 多相机 Group | GMK 景深融合 |
|---|---|---|
| "多"指什么 | 多台**物理相机** | **同一台相机多次位移** |
| 怎么组合 | `GroupFrame` Map | 下位机 `capture_id` / `fusion_id` |
| 谁做合并 | 本库 `CameraGroup` | **板端 / IB 引擎** |
| 产物 | N 台各一张 | **一张融合图** |
| 需要同步？ | 需要 | **不需要**（本就是依次拍） |

**结论**：融合发生在"帧被驱动交出来之前"，Group 只当它是台普通单帧相机。所以 GMK 走的是**单相机 + 多次位移**，与 CameraGroup 无关。

## 5.7 GMK IS 相机的特殊性

`INS-CHVS-363-10GM-BB` 内置 4 路 LED 频闪：

- 需要 `LightSourceMode` + 4 个分路曝光；
- 必须配 `SplitByRow` 拆图 + `SplitByRowIndex` 分流；
- **必须走 `External`**，否则 4 路时序对不上。

## 5.8 已知缺陷

| 缺陷 | 位置 | 后果 |
|---|---|---|
| 帧队列满时**静默丢整组**，无事件 | `CameraGroup.cpp:441` | 丢帧不可观测 |
| 超时残帧推送**无标记** | `CameraGroup.cpp:512` | 调用方需自行判空 |
| 无 `frameId` | — | 快触发会把两批拼成一组 |

---

# 第三编 业务：GMK 系统怎么工作

# 第 6 章 GMK 系统总览

## 6.1 业务目标

对被测件做**景深融合**：Z 轴逐层步进拍摄，每层一张，最终堆叠出一张全焦图，用于后续检测（现场以光模块类样品为主）。双相机方案则是左右各一路，各自融合。

## 6.2 部署形态

```
上位机（Windows，lpcGMKWin 插件，TCP V3 客户端）
        │ 9500 控制 / 9501 左 / 9502 右
板端（Windows 工控机，GMKdemo.exe）
        ├─ LPC 相机库 → 大恒相机（进程内）
        ├─ TCP → PLC 6000/6001（电机、光源）
        └─ IB 引擎（进程内，CPython）
```

**板端为什么是 Windows**：二期方案把板端从 Linux/RK3588 迁到 Windows，直接复用 LPC 相机库与大恒 SDK，代价是实时性不如 RTOS——所以运动节奏全部交给 PLC 内部执行，板端只响应事件。

## 6.3 四条外部边界

| 边界 | 协议 | 板端角色 | 代码 |
|---|---|---|---|
| 相机 | 进程内 C 调用 + 回调 | 调用方 | `L*` 函数 |
| PLC | TCP ASCII 文本，`;` 分帧 | 客户端 | `motion/MotorPlc.*` |
| 上位机 | TCP V3 二进制 | 服务端 | `server/GmkServer.*` |
| IB 引擎 | 进程内 C++ + CPython | 调用方 | `ib/IBFusionRunner.*` |

## 6.4 配置文件 GMKdemo.ini

放在 exe 同目录，随构建自动部署。所有端点可配：

```ini
[Server]                 ; 板端作为服务端
bind_ip=0.0.0.0
control_port=9500
left_port=9501
right_port=9502

[Plc]                    ; 板端作为客户端
host=192.168.10.200
port=6000
timeout_ms=5000

[Camera]
device=DaHeng (V2.6.2608)
galaxy_sdk=GalaxySDK     ; 相对 exe 或绝对路径
capture_timeout_ms=5000

[Fusion]
ib_path=...\IB_3.8.86_x64_softEnc
task_path=...\ronghe.task
```

## 6.5 启动流程

1. 启动 `GMKdemo.exe`（首次读取 `GMKdemo.ini`）；
2. 相机：`枚举驱动` → 选 `DaHeng (V2.6.2608)` → `枚举相机` → `添加` → `打开相机` → `开始采集`；
3. 右侧「相机参数」树可读写全部参数（TriggerMode / ExposureTime / Gain / Width …），与 QtDemo 一致；
4. 融合：默认按 ini 自动加载，也可用「选择 IB 目录 / 加载融合 Task」重指定；
5. `启动 Server`：按配置监听控制口与左右数据口。

## 6.6 抓拍模式

| 场景 | 请求字段 | 板端行为 |
|---|---|---|
| 单张 | `kind=snapshot,count=1,outputs=originals` | 左右各 1 帧原图 |
| 同步融合 | `kind=focus_stack,count=N,motion_mode=continuous\|step,outputs=originals\|fused\|both` | 左右各 N 帧，单次/逐步移动 |
| 异步融合 | `motion_mode=async,left_start_step,right_start_step,left_count,right_count` | 左右各自起始步与张数 |

**输出选项**：

- `originals`：仅原图（0x0011，边采边传）
- `fused`：仅融合图（0x0022，每路 1 张）
- `both`：两者都回
- 结束：每路 `CHANNEL_DONE`（0x0024）+ 控制通道 `TASK_DONE`（0x0023）

其它可调参数走 `0x0006/0x0007 GET/SET_PARAMETERS`：`exposure_ms / gain_db / frame_count / step_mm / motion_mode / direction / left_count / right_count / left_start_step / right_start_step / light_percent`。

> ⚑ 文档与实现不一致，**以代码为准**：`GMKWin.md` 说 `async` 支持左右独立张数，代码已强制 `right_count = left_count` 并打警告（见 Q-02）。

---

# 第 7 章 板端 GMKdemo 内部结构

## 7.1 目录职责

> ⚑ 按当前代码复核（口径：`*.cpp` / `*.h` / `*.ui`）：

| 目录 | 文件 | 行数 | 职责 |
|---|---|---|---|
| `GMKBoard.cpp/.h` | 2 | 3,426 | ★ 核心业务（无 Qt）：任务状态机、抓拍、推流、融合调度 |
| `ui/` | 15 | 3,664 | Qt 界面（`GMKBoardWindow.cpp` 单文件 2,626） |
| `ib/` | 3 | 1,425 | IntelliBlink 融合封装 |
| `motion/` | 2 | 933 | PLC ASCII 协议 |
| `server/` | 3 | 922 | TCP V3 服务端 |
| `config/` | 4 | 740 | BoardConfig（ini）+ GalaxyEnv（大恒环境） |
| `util/` | 5 | 566 | Log / WorkerThread / ImageTransform |
| `main.cpp` | 1 | 22 | 程序入口 |
| **合计** | **35** | **11,698** | |

> ⚑ 原稿记"39 文件 / 8,001 行 / 425 KB"，当前实测 35 文件 / 11,698 行。差异来自统计口径（本表含 `.ui`）与代码增长。

**开发边界**：只应改 `src/GMKdemo/` 与 `tpvs17/GMKdemo/`；`CameraManager`、`Core` 属于库，改之前要评估回归面。

## 7.2 依赖方向

实测无循环，单向：

```
util（零依赖）
  ↑
config / motion / server      （仅依赖 util）
  ↑
ib                            （依赖 config + util）
  ↑
GMKBoard
  ↑
ui                            （依赖 config + util）
```

**意义**：`util` 可以被任何人依赖，`GMKBoard` 是收敛点；想在 UI 里直接调 PLC，就会破坏这个方向。

## 7.3 核心数据结构

| 结构 | 含义 |
|---|---|
| `BoardParams` | 界面层参数（`std::wstring`），`SET_PARAMETERS` 直接写入 |
| `CaptureRequest` | 电机请求（内部中转），含 `travelMm` / `moveIntervalMs` |
| `CaptureTask` | 抓拍任务（运行时状态），含计数器与完成标志 |
| `ProcessingOptions` | 融合参数快照（每组抓拍冻结一份） |
| `ProcessingSettings` | 输出参数：`scale` / `targetWidth` / `targetHeight` / `cropPosition` / `previewFps` |
| `FrameJob` | 处理队列单元；`jobId` 空 = 预览帧，非空 = 抓拍帧 |
| `FusionInputFrame` | 送算法的输入，**时间戳只透传不改** |
| `FusionResult` | 算法输出：`splitImages[4]` + `topLefts[4]` + `realtimeImage` + `depthFusionImage` + `needStop` + `clarity` |
| `fusionOutputDiagnostic` | 融合图缺失时的**取证字符串**（task id + 原因 + 实际输出清单） |

## 7.4 一帧的旅程

```
PLC 推 "3" → TriggerFrame → LSnapCamera → 相机驱动回调（驱动线程）
  → OnLpcFrame → EnqueueOriginalsRaw（从池取复用缓冲，1 次 memcpy）
  → ScheduleFrame → fusionQueues_[1]/[2]（左右独立）
  → FusionLoop(1)/(2) → RunFrameJob
  → IBProcessingPipeline::Process（拆图 → 4 路并行融合 → 合成）
  → PublishOutputs（TCP / UI / 存图）
  → MaybeFinishChannel → MaybeFinishTask
  → WaitChannelDrained → CHANNEL_DONE + TASK_DONE
```

一次 CAPTURE 跨 **5 类线程**：`ControlLoop` → 抓拍线程（detach）→ 相机回调线程 → `FusionLoop` → `SenderLoop`。

## 7.5 输入链路的七步优化

相机回调线程上执行的七步，每一步都在砍开销：

| 步骤 | 优化点 |
|---|---|
| 参数校验 | 早退，避免无意义拷贝 |
| `needstop` 检查 | **在拷贝前**丢弃 |
| 查询任务 | 小锁，**不包含 memcpy** |
| 预览限频 | **在拷贝前**丢弃（空闲 20fps 只留 5fps） |
| 取池缓冲 + memcpy | 池复用避免 malloc，**只拷一次** |
| 链路未就绪降级 | 显示原图但不跑算法 |
| `std::move` 入队 | O(1) 交棒 |

**核心**：只保留一次不可避免的 `memcpy`（驱动会复用 `fd->pData`），其余靠缓冲池、提前丢弃、移动语义消除。

## 7.6 任务状态机

| 标志 | 作用 |
|---|---|
| `taskRunning_` | 五重作用：屏蔽新任务、状态上报（busy/idle）、取消信号、丢过期数据（配合 `jobId`）、保护资源 |
| `motionRunning_` | PLC 还在动；`MaybeFinishTask` 必须等它清零才发 `TASK_DONE` |
| `plcSequenceMutex_` | 串行化整个抓拍的 PLC 交互，期间手工指令被拒 |
| `softTriggerRejectStreak_` | 连续被拒 5 次后本任务内不再调 `LSnapCamera`（外触发自适应） |

**`EndMotionRun()` 是唯一允许发 `TASK_DONE` 的位置**；`FailTask` 与 `Cancel` 都用 `exchange(false)`，保证结束处理只执行一次。

## 7.7 IB 融合的四层封装

| 层 | 类 | 数量 | 职责 |
|---|---|---|---|
| 引擎层 | `lpbengine.dll` + CPython | 1 | 跑 Python 算法 |
| 服务层 | `IBServiceRuntime` | **1（全局）** | `Create` / `SetUp` / `TearDown` 引擎开关 |
| 任务层 | `IBFusionRunner` ×5、`IBStopsnapRunner` ×1、`IBFlightRunner` ×1 | 7 | 单个 task 的执行 |
| 编排层 | `IBProcessingPipeline` | **1（每链路）** | 组织 + 并行调度 + 合成 |

**为什么是 4 个融合实例**：一张大图被 `拆图.task` 切成 **4 块**，每块需要独立实例（同 task 实例内部有帧累积状态，不能并发处理不同区域），故 4 实例 + 4 线程一一绑定。

> ⚑ 风险：块数硬编码 4，改块数要改多处数组（见 B-04）。

**四条业务路径**：

| 路径 | 入口 | 说明 |
|---|---|---|
| 普通融合 | `Process()` | 拆图 → 4 路融合 → 合成，末帧出融合图 |
| 停拍评分 | `EvaluateStop()` | **不拆图不融合**，按 Clarity 选最优帧 |
| 飞拍评价 | `EvaluateFlight()` + `Process()` | 判定 `FastCapture` → 发 `PIMRUN`；融合延迟到运动结束 |
| 空闲预览 | `Configure(fusion=false)` + `Process()` | 只拆图不融合，**不发 Reset**（避免干扰抓拍） |

**停拍评分与拆图融合是 `if / else` 互斥的**（`GMKBoard::RunFrameJob`）。

## 7.8 缓冲池与延迟回收

- 单帧 11264×9200 MONO8 ≈ **98.8 MiB**，分配一次要 20–24ms，而帧间隔仅 8ms → **必须复用**。
- 每通道池上限 `kMaxPooledInputBuffers = 3`。
- 归还时 `refcount > 1` 只进 `pending`，等所有引用释放才升级到 `free`——**延迟回收**，因为图可能同时被融合、预览、存图引用。
- 左右通道所有资源**各有一把锁，绝不共用**：`fusionMutexes_[1..2]`、`inputPools_[1..2].mutex`、`leftMutex_` / `rightMutex_`、`leftQueue_` / `rightQueue_`。共用会导致先到的一路吃掉全部资源，另一路每帧重新分配 130MB。

---

# 第 8 章 PLC 通信链路（对下）

> 本章合并原稿第 2 章（2.1–2.8）、19.1（细化）与第 5 章（EL/MMC），三处讲的其实是同一条链路。

## 8.1 角色与端点

| 项 | 内容 |
|---|---|
| 通信方式 | **TCP 长连接**，板端作**客户端** |
| 端点 | EL 单轴 `:6000`；MMC 轴1 `:6001`（同 IP `192.168.10.200`） |
| 配置 | `GMKdemo.ini` 的 `[Plc]` 段 |
| 协议 | **ASCII 文本**，分号结尾 |
| connect 调用 | 全项目**仅此一处**（`MotorPlc.cpp:51`），其余 socket 全是 listen/accept |

**模块**：

| 模块 | 角色 |
|---|---|
| `PDUMotionWrapper` | 独立静态库，TCP 客户端封装（`src/PDUMotionWrapper`） |
| `MotorPlc` + `PlcTcpTransport` | GMKdemo 内的 PLC 适配层（`src/GMKdemo/motion`） |
| `CameraExOCT` | 使用 `PDUMotionWrapper` 的相机插件 |

## 8.2 报文格式

**请求无尖括号，应答带尖括号**：

```
请求：CHK,1;                 SETPOS,1,3,12.500;       INTERVAL,1,14,0.040,1,1;
回答：CHK,1,0011,12.345;     ← 状态字段为【十六进制】
      CMD,1,1;                ← 1 = Success，2 = Failure
```

两种等价写法在代码里都出现过（逗号分隔与空格分隔），**发送只认逗号**，接收时容错空格——这是原稿"认知修正"里的一条，排障时最容易踩。

## 8.3 命令表

| 类别 | 命令 |
|---|---|
| 回零 / 检查 | `HOME` `CHK` |
| 使能 / 复位 | `SEVERON` `RST` |
| 单步 / 连续 | `SSP` `GSP` `INCSTART` `STOP` `RUNSTEP` `RunContinues` `RUNCSTEP` `NeedStop` |
| 位置 / 间隔 | `SETPOS` `INTERVAL` `PIM` |
| 光源 | `LightON` `LightBRISET` `LightPOSSET` `TRIGON` |

## 8.4 状态流与"循环在谁手里"

状态码语义：`1` = 就绪/到位，`3` = **该拍了**，`4` = 结束，`2` = 失败。

**关键认识：连续模式下 14 次循环在 PLC 内部执行。**

```
板端：INTERVAL 1 14 0.04   → 一次性下发计划
      RunContinues 1 1     → 开跑
之后：只被动响应 14 次 "3"
```

板端**不参与逐点调度**——8ms 帧间隔下，板端往返通信根本跟不上。

**三种运动模式的循环位置不同**：

| 模式 | 循环在哪 | 板端角色 |
|---|---|---|
| 连续（`RunContinues`） | **PLC 内部** | 只响应 `3` |
| 步进（`RunStepCapture`） | 板端 `for` | 每步发 `SETPOS` + `RUNSTEP` |
| 停拍（`RunCStepCapture`） | 板端 `for` | 每步等停稳再拍 |

**相机拍照由 PLC 的 `3` 事件驱动，不是板端定时。**

## 8.5 懒连接与断链判定

**构造时不建连**，只保存 host/port/超时；第一次发命令才 `EnsureConnected()`：

```cpp
if (sock_) return true;                       // 已连，直接复用
getaddrinfo → socket() → connect()            // 否则建连
```

| 行为 | 说明 |
|---|---|
| 失败不缓存 | 下次命令重新尝试，无需手动重连 |
| 接收超时**不断链** | 只返回空串，socket 保留复用（`MotorPlc.cpp:257`） |
| 真正断链 | 仅 `send` 失败、`recv` 错误、流超时/取消、看门狗 `Reconnect()` |

## 8.6 带超时的非阻塞 connect

Windows 默认 `connect` 到不可达地址会阻塞约 **21 秒**，必须非阻塞 + `select`：

```cpp
u_long nonBlocking = 1;
::ioctlsocket(s, FIONBIO, &nonBlocking);        // ① 切非阻塞
::connect(s, addr, addrLen);                    // ② 立即返回 WSAEWOULDBLOCK
::WSAGetLastError() == WSAEWOULDBLOCK           // ③ 握手进行中（不是错误）
::select(0, nullptr, &writeSet, &errSet, &tv);  // ④ 等可写/出错/超时
::getsockopt(s, SOL_SOCKET, SO_ERROR, ...);     // ⑤ 二次确认 SO_ERROR == 0
u_long blocking = 0;
::ioctlsocket(s, FIONBIO, &blocking);           // ⑥ 切回阻塞
```

**关键**：`select` 返回可写**不代表成功**，必须 `SO_ERROR == 0` 二次确认。

## 8.7 五种收发模式

`GMKBoardWindow.cpp` 的 `makeMotor` 是全工程**唯一**的 `PlcTcpTransport` 实例化点，被调用两次（EL / MMC），各得一条独立 socket。

| 入口 | 方法 | 解决的问题 |
|---|---|---|
| `transport_` | `operator()` | 普通请求-应答；`streaming_` 时主动拒绝 |
| `streamTransport_` | `StreamLines()` | 一请求多应答；接收时**不持锁**，给急停让路 |
| `sendOnly_` | `SendOnly()` | 流运行期间急停必须能送达，只 `::send` |
| `abortTransport_` | `AbortStream()` | 只置标志位，流自己 drain/close，避免 RST 丢掉刚发的 STOP |
| `reconnectHook_` | `Reconnect()` | 看门狗强制断链，下次命令重建 |

**为什么必须五个分开**：若只有一个 `transport_`，采集期间每秒一次的 `CHK` 轮询会把状态流里的 `3`/`4` 抢走，导致应答错位、任务提前 `done`。**加锁解决不了，只会卡死 UI。**

## 8.8 并发保护

| 场景 | 处理 | 位置 |
|---|---|---|
| 流运行中普通命令 | `if (streaming_.load()) return {};` | `operator()` |
| 流运行中第二个流 | `streaming_.exchange(true)` 失败返回 false | `StreamLines()` |
| 流运行中急停 | `SendOnly` 放行 | `StopAndAbort()` |
| 状态轮询 | 调用方先 `IsStreaming()` 跳过（双保险） | `QueryStatusAsync()` |

## 8.9 线程模型

**UI 线程从不直接碰 socket**，全部经 `WorkerThread` 串行队列投递：

| 执行体 | 用途 |
|---|---|
| `plcWorker_` | 手动命令、状态轮询、链路看门狗 |
| `captureWorker_` | UI 抓拍（抓拍慢，与手动命令分道） |
| detach 线程 | TCP 远程抓拍 |
| `watchdogThread_` | 任务看门狗 |

**为什么要分道**：抓拍会阻塞整个扫描过程；若与手动命令共用队列，急停按钮会排到扫描之后，界面看起来像死机。

## 8.10 EL 与 MMC

| 项 | EL | MMC |
|---|---|---|
| 轴数 | 单轴 | 双轴（轴1、轴2） |
| 端口 | 6000 | 6001 / 6002 |
| 本工程链路 | 1 条 | 1 条（仅轴1接入代码） |
| 抓拍 | `SETPOS×N` → `SSP` → `RUNCONTINUES` | 同步（两轴同路径）/ 异步（左右起点不同） |

运行时由 `activeMotor`（`"el"` / `"mmc"`）选择；**急停会同时下发到两者**。

## 8.11 一次连续抓拍的时序全景

```
上位机 CAPTURE
  → ControlLoop 收帧 → PrepareTaskForCapture（同步，可能报错）
  → StartCapture（detach 异步，避免卡死控制通道）
  → SETPOS（起始前的位置）→ INTERVAL 1 14 0.04 → RunContinues 1 1
  → PLC 内部循环 14 次：
        到位 → 推 "3" → 板端 LSnapCamera → 相机回调 → 融合
  → PLC 推 "4"（结束）→ 电机回位 → EndMotionRun → TASK_DONE
```

**两个易错点**：

- `PrepareTaskForCapture` 必须**同步**（失败要立刻在 RESPONSE 里告诉上位机），`StartCapture` 必须**异步**（否则连 CANCEL 都收不到）；
- 顺序是 `PrepareTaskForCapture → StartCapture`，不是反过来。

---

# 第 9 章 上位机通信链路（对上）

> 本章合并原稿第 3 章（3.1–3.6）、19.2（细化）与第 6 章（客户端），三处讲的是同一条链路的两端。

## 9.1 三端口

| 端口 | 用途 | 上限 |
|---|---|---|
| 9500 | **控制通道**，JSON 请求/应答 | ≤ 1 MiB |
| 9501 | **左相机数据通道**，二进制图像 | ≤ 256 MiB |
| 9502 | **右相机数据通道**（单相机方案不监听） | ≤ 256 MiB |

图像与控制走**不同 TCP 连接**，避免大图阻塞指令应答。

## 9.2 协议来源与项目改动

**这不是项目自创协议**，是外部规范 `TCP_V3.md` 定义的。项目做了四处实质改动：

| 改动 | 内容 | 后果 |
|---|---|---|
| CRC 禁用 | 两端 `payloadCrc32` 恒写 0 | 无完整性校验，依赖 TCP |
| `kMaxPayload` 64 MiB → 256 MiB | 原值装不下 98.8 MiB 单帧 | **曾导致数据通道静默断开**（客户端每帧报 invalid header） |
| 扩展双数据通道 | 9501/9502 + `ImageHeaderV3.channel` | 从单相机扩展为双相机 |
| 新增消息 | `MOTOR_AVAILABLE`（0x2C） | 支持"采图完成即交还电机"的异步运动 |

> ⚑ 风险：`kMaxPayload` 在 `GmkProtocol.h:56` 与 `CameraGMKWin.cpp:84` **各定义一次**，只靠注释同步。改单边会让数据通道**静默失效**（见 S-03）。

## 9.3 帧格式

**帧头 `FrameHeader` 固定 24 字节**（`static_assert` 校验）：

| 字段 | 含义 |
|---|---|
| `magic` = `0x4D504F31` | 线上字节 `31 4F 50 4D` |
| `version` = 3 | |
| `frameType` | 1=请求 2=应答 3=数据 4=通知 |
| `msgId` / `seq` | 消息号 / 序号 |
| `payloadLen` / `payloadCrc32` | 载荷长度 / CRC（**实际恒为 0**） |
| `reserved` | 必须为 0 |

**图像头 `ImageHeaderV3` 固定 72 字节**，后接原始 MONO8：

```
frameIndex / totalCount / width / height / pixelFormat(4=MONO8) / stride
timestampUs / imageSize / jobId[24] / channel(1=左 2=右) / parameterRevision
```

> ⚑ `timestampUs` 用 `steady_clock`（开机以来微秒），**不是墙钟时间**。下游若当日期用会错（见 S-12）。

## 9.4 六条返回链路

| 消息 | 通道 | 时机 |
|---|---|---|
| `RESPONSE` | 控制口 | 每个请求 |
| `PROGRESS` | 控制口 | 每收到一帧 |
| `CAPTURE_DONE` / `MOTOR_AVAILABLE` | 控制口 | 所有相机帧收齐 |
| `TASK_DONE` | 控制口 | 整轮结束（含电机回位） |
| `IMAGE_DATA` / `FUSED_IMAGE_DATA` / `CHANNEL_DONE` | **数据口** | 每帧处理完 |

**为什么控制口会"超车"**：控制消息小，数据图大。实测 142MB 融合图比 `TASK_DONE` 晚 15ms 到达，客户端统计成 0 张。解决靠 9.7 的 `WaitChannelDrained`。

## 9.5 线程模型

**每个端口独立线程**，数据通道收发分离：

```
Start()
 ├─ ListenerLoop(9500, ch=0) → accept → ControlLoop 线程
 ├─ ListenerLoop(9501, ch=1) → accept → SenderLoop 线程 + DataLoop 线程（分离）
 └─ ListenerLoop(9502, ch=2) → accept → SenderLoop 线程 + DataLoop 线程（分离）
```

| 线程 | 数量 | 创建时机 | 阻塞点 | 唤醒源 |
|---|---|---|---|---|
| `ListenerLoop` | 3 | `Start()` | `accept()` | 客户端连接 |
| `ControlLoop` | 1 | 客户端连控制口 | `recv()` | 上位机命令 |
| `DataLoop` | 2 | 客户端连数据口 | `recv()` | `BIND_DATA` |
| `SenderLoop` | 2 | 客户端连数据口 | `cv.wait()` | 融合线程入队 |

**三个细节**：

1. 四类线程**全是阻塞模型**（thread-per-connection），没有事件循环；
2. `SenderLoop` **不是启动时创建的**，是客户端连上数据口时由 `ListenerLoop` 的 accept 分支创建；
3. `SenderLoop` **不判断端口**，它读 `q->socket`，而这个 socket 由监听 9501/9502 的 `ListenerLoop` 写入。**"是哪个端口"在 bind 时就定了，之后只传句柄。**

**为什么收发分两个线程**：一条连接上收和发可能同时阻塞——`DataLoop` 卡在 `recv` 时，`SenderLoop` 仍要能往同一 socket 写图。

## 9.6 发送队列与背压

单帧 98.8 MiB，必须异步发送：

| 设计 | 作用 |
|---|---|
| **异步队列** | `SendImageFrame` 只 move 进队列就返回，相机回调零阻塞 |
| **缓冲池复用** | `spare` 池避免每帧重新分配（24ms → 12ms） |
| **背压 = 丢最旧帧** | 6 GiB/通道预算，超限 `pop_front` 并计数 |
| **`inFlight` 计数** | 已出队但仍在写的帧也算未排空 |
| **写锁保护** | 应答与图帧不能交错，否则帧头错位导致客户端丢通道 |

## 9.7 跨口超车与 WaitChannelDrained

**`WaitChannelDrained` 的意义**：`TASK_DONE` 前必须等队列空 **且 `inFlight == 0`**，否则大图会在 `TASK_DONE` 之后才到，客户端计数为 0。

**为什么 `inFlight` 必需**：只看 `frames.empty()` 会误判——帧已被取出、但仍在 `send()` 里。

`SenderLoop` 发完后 `--inFlight` + `notify_all()`，唤醒的是 `WaitChannelDrained`（**不是** `SenderLoop` 自己，它不可能等自己）。

## 9.8 性能调优

```cpp
int sndBuf = 8 * 1024 * 1024;                    // 默认 64KiB 只跑到 ~720 MiB/s
::setsockopt(client, SOL_SOCKET, SO_SNDBUF, ...);
::setsockopt(client, IPPROTO_TCP, TCP_NODELAY, ...);
```

## 9.9 上位机客户端插件

| 文件夹 | 说明 |
|---|---|
| `src/CameraGMKWin` | 新版客户端（TCP V3），`kMagic` / `kVersion` 与板端完全一致 |
| `src/CameraGMK` | 旧版 / 另一路客户端 |

对上位机软件来说它就像"一台双目 GMK 相机"——打开、抓图、收图，底层实为 TCP。枚举出两个逻辑设备：`host|port|left`、`host|port|right`。

## 9.10 会话与安全限制

**外部上位机可用性**：任意程序只要实现 TCP V3 即可取图，但——

| 限制 | 后果 |
|---|---|
| 控制连接仅 1 个 | 后连者踢掉先连者 |
| 每个数据通道仅 1 个 | 同上 |
| 无认证 / 无加密 | 内网可用，公网危险 |
| `payloadCrc32` 恒为 0 | 无完整性校验（依赖 TCP） |

即：**技术上通用，实际上同一时刻只支持一个上位机会话**。

---

# 第四编 深化：并发、质量与结论

# 第 10 章 并发与线程模型

> 本章合并原稿第 4 章（拒绝矩阵）、19.3（锁与优先级）、2.7/2.8（PLC 侧并发）、3.4（服务端线程）。原来四处各讲一半，现在按"规则 → 机制 → 案例"重排。

## 10.1 两套并发问题

| 层面 | 问题 | 手段 |
|---|---|---|
| **板内** | 多线程抢同一批资源（PLC 链路、融合队列、缓冲池） | 互斥量 + 优先级 + 拒绝 |
| **板间** | 端口间消息乱序（控制口超车数据口） | 队列 + `inFlight` + `WaitChannelDrained` |

两者容易混：板内的锁**解决不了**板间的乱序，板间的等待也**不能**持板内的锁（会把相机回调拖住）。

## 10.2 优先级规则

```
抓拍（长任务，占锁） > 手动命令（try_lock，抢不到就拒） > 状态轮询（抓拍时跳过） > 急停（不受锁限制）
```

- `plcSequenceMutex_` 是 `timed_mutex`，**抓拍全程持有**；
- 手动命令用 `try_lock` 抢占，失败即**拒绝并记日志**（不静默丢弃）；
- 急停走 `SendOnly`，**不参与排队**。

## 10.3 拒绝矩阵

| 场景 | 结果 |
|---|---|
| 任务运行中又来 CAPTURE（任何通道） | `1003 busy`（`GMKBoard.cpp` 的 `taskRunning_` 检查） |
| 抓拍中手动点动 / 回原点 | 忽略 + 日志，UI 不卡（`TryLockPlcSequence`） |
| 流运行中状态轮询 `CHK` | 跳过 / 拒绝 |
| 流运行中第二个状态流 | `return false`（`streaming_.exchange`） |
| **流运行中急停 / 停拍** | ✅ **放行且优先**（`SendOnly`） |

**为何选择拒绝而不是排队**：抓拍期间 `SETPOS` 与 `RUNCONTINUES` 之间不能插入任何命令，否则运动配方错乱；而排队的手动命令在执行时用户早已忘记，行为会变得莫名其妙。

## 10.4 三层异步防线

| 机制 | 保护谁 | 不做的后果 |
|---|---|---|
| `captureWorker_` | UI 线程 | 点抓拍后界面"未响应" 30 秒 |
| `plcWorker_` | UI 线程 | PLC 不可达时界面冻结 |
| `GMKBoard::StartCapture` detach | `ControlLoop` 线程 | TCP 收不到 CANCEL / 心跳 |

三者**正交**：前者保护 UI，后者保护控制通道，不重复。

## 10.5 锁顺序与 RAII

- `BeginMotionRun` / `EndMotionRun` **只是状态标记，不驱动 PLC**；真正驱动在 `RunContinuousMotor` / `RunStepCapture` / `RunCStepCapture`。
- 用 **RAII Guard** 保证 `EndMotionRun` 一定执行（异常也清）。
- `motionGuard` 声明在**外层作用域**，保证**先解锁 PLC 再 `EndMotionRun`**——否则 `TASK_DONE` 触发新命令时死锁。

## 10.6 锁外操作原则

`ScheduleFrame` 锁内只做 `push_back(std::move(...))`，归还缓冲放锁外。

**放锁内虽不会死锁**（`AcquireInputBuffer` 不反向取队列锁，无 AB-BA 环），但会让"等池锁"传染成"持队列锁"，卡住相机回调；磁盘慢时正反馈恶化。

**这是一条通用规则**：锁内只做数据结构操作，I/O、分配、回调一律放到锁外。

## 10.7 取消机制

- 取消靠 `taskRunning_` 广播：所有长循环用 `cancelled` lambda 检查，这是**跨线程的取消通知**。
- `taskRunning_` 五重作用：屏蔽新任务、状态上报、取消信号、丢过期数据（配合 `jobId`）、保护资源。
- `FailTask` 与 `Cancel` 都用 `exchange(false)`，保证结束处理**只执行一次**。

## 10.8 内存与拷贝

| 认知 | 说明 |
|---|---|
| `std::move` 本身不移动任何东西 | 只是类型转换；真正移动由移动构造完成 |
| `cv::Mat` 的移动只是偷指针 | `data` 指针 + `u` 引用计数，**像素数据原地不动** |
| 缓冲归还的 `refcount > 1` 是延迟回收 | 图可能被融合、预览、存图同时引用 |
| 每通道池上限 3 | 每帧约 114 MB，上限即内存预算 |

## 10.9 一个死锁案例

**回调持锁调用户代码 → 锁升级死锁**（`CameraCtrlCallback.cpp:20`）：用户在图像回调里注销自己，会去拿同一把锁 → 死锁。

**规避**：回调里不要做注册/注销/关闭相机这类"反向操作"，需要的话投递到自己的队列异步处理。

---

# 第 11 章 质量与风险台账

> 原稿有四份问题清单（第 7、11、14、19.6 章），共 40 余条，重复项已合并。本章按层分类并统一编号，编号在全书可引用。

**分级**：`P0` 阻塞主链路 / `P1` 有功能缺陷或不可观测 / `P2` 可维护性 / `P3` 记录备查。

## 11.1 环境与部署

| 编号 | 级别 | 问题 | 影响 | 建议 |
|---|---|---|---|---|
| E-01 | P0 | IB 目录 `E:\GMKTEST\IB_3.8.86_x64_softEnc` 不存在 | **主链路完全阻塞**，所有抓拍被 `kErrCapability` 拒绝 | 索取 IB 3.8.86 完整包（含 4 个 `.task`） |
| E-02 | P1 | `GMKGalaxySdk` / `GMKLPVBin` / `GMKIBDir` 未定义 | 构建后不部署 GalaxySDK / lpv / IB 运行库 | 在 `salticidae.props` 或 vcxproj 中定义 |
| E-03 | P1 | `plugins/` 缺 `GxIAPICPPEx.dll` | 大恒驱动加载失败 | **已手动补齐**，建议加部署目标 |
| E-04 | P2 | `QtInstall` 硬编码 `C:\Qt\Qt5.9.4\...` | 他人编译失败 | 改 `$(QTDIR)` 或 `Exists()` 条件回退 |
| E-05 | P3 | `runner17\data\` 与 `runner17\salticidae\data\` 重复 | 少量冗余（102 文件） | 属 Core 工程配置，可忽略 |
| E-06 | P2 | `3rdparty` SDK 实体需内网获取 | **无法自举** | 补 `bootstrap.ps1` 做校验 |

## 11.2 框架层

| 编号 | 级别 | 问题 | 位置 | 影响 |
|---|---|---|---|---|
| F-01 | P1 | 回调持锁调用户代码 → **锁升级死锁** | `CameraCtrlCallback.cpp:20` | 用户在回调内反注册即死锁 |
| F-02 | P1 | 帧队列满时**静默丢整组**，无事件 | `CameraGroup.cpp:441` | 丢帧不可观测 |
| F-03 | P2 | 超时残帧推送**无标记** | `CameraGroup.cpp:512` | 调用方需自行判空 |
| F-04 | P1 | `_CloseCamera` **变量遮蔽** | `CameraCtrl.cpp:537` | 停流失败不向上传播 |
| F-05 | P1 | 配置保存先 `remove_all` 再重建 | `CameraManager.cpp:710` | 误删目标目录内其他文件 |
| F-06 | P2 | 配置 `version` 字段写了不校验 | `CameraManager.cpp:731` / `:1098` | 版本兼容无保护 |
| F-07 | P2 | 进程 PATH 被永久污染（只增不减） | `CameraLibraryHelper.cpp:226` | 重复 `LInit` 无限增长 |
| F-08 | P3 | `LPC.h:5` `#include "LCameradefine.h"` **大小写错误** | `include/LPC.h` | Windows 能过，Linux/Mac 会失败 |

> 原稿把 F-07、F-08 各写了两遍（框架章 + 风险章），此处合并。

## 11.3 驱动层

| 编号 | 级别 | 问题 |
|---|---|---|
| D-01 | P2 | `CameraAutomv` 设备名标 **"CL" 实为 GigE**、SN 写死、`IStartCamera` 与 `ISnapCamera` 同实现（`CameraAutomv.cpp:94`） |
| D-02 | P2 | `CameraArtray` Bayer 分支 `new unsigned char[]` **无对应 delete**（`CameraArtray.cpp`） |
| D-03 | P2 | `CameraItekCL` 与 `CameraItekCLV1125` 源码 **99% 重复** |
| D-04 | P3 | `CameraSelftest.cpp` 单文件 6,020 行 |
| D-05 | P3 | `CameraDefinePrivate.cpp` 是 0 KB 空文件 |

## 11.4 板端业务

| 编号 | 级别 | 问题 | 位置 | 建议 |
|---|---|---|---|---|
| B-01 | P1 | `GMKBoard` 上帝类（3,426 行、6 类职责、100+ 成员） | `GMKBoard.cpp` | 优先拆分 `HandleControl`（540 行 switch） |
| B-02 | P2 | `GMKBoardWindow.cpp` 单文件 2,626 行 | `ui/` | 按功能拆分面板类 |
| B-03 | P2 | `Process()` 里 `clarity` 收集是死代码 | `IBProcessingPipeline.cpp:209` | 停拍不走 `Process`，可清理 |
| B-04 | P2 | 融合实例数量硬编码 4 | 多处数组 | 提取为常量或按 task 配置 |
| B-05 | P2 | `motionMode=async` 语义冗余 | `GMKBoard.cpp:1037` | 与 `continuous` 走同一路径，确认是否废弃 |
| B-06 | P2 | `IBServiceRuntime::Shutdown()` 缺失导致进程残留 | `main.cpp` 退出流程 | IB 持有 CPython 与线程池；**疑似与现场 `tpshell` 关不掉相关** |
| B-07 | P3 | `g_serviceInitialized` 非 atomic | — | 当前单线程调用，安全 |
| B-08 | P3 | 日志中文乱码（`浣跨敤` = `使用`）、三个 `.ui` 疑似乱码 | `util/Log.cpp` | GBK/UTF-8 不一致，统一 UTF-8 |

> B-08 合并了原稿两处（日志乱码 + `.ui` 乱码），同一根因。

## 11.5 协议与服务器

| 编号 | 级别 | 问题 | 位置 | 影响 / 建议 |
|---|---|---|---|---|
| S-01 | P1 | 控制口重连竞态 | `GmkServer.cpp:254-255` | `controlSocket_` 无锁读写；退出时**无条件**清连接标志，会冲掉新连接；建议参照数据口的 `if (socket == socket)` |
| S-02 | P1 | 控制口句柄泄漏 | 同上 | `ControlLoop` 退出未 `closesocket`，每次重连漏一个 |
| S-03 | P1 | `kMaxPayload` 双端各自定义 | `GmkProtocol.h:56`、`CameraGMKWin.cpp:84` | 改单边 → 数据通道**静默失效**；建议共享头或握手校验 |
| S-04 | P1 | `Start()` 返回值不可信 | `GmkServer.cpp:96-107` | bind 失败仅打日志，UI 仍显示"运行中"；建议同步 bind 后再起线程 |
| S-05 | P2 | `DataLoop` 停止超时无告警 | `GmkServer.cpp:661-663` | 轮询 1 秒后继续 `WSACleanup()`，线程未退可能访问已销毁对象 |
| S-06 | P2 | 控制/数据连接单会话，后连踢前连 | `GmkServer.cpp:172`、`:208` | 需多上位机时增加会话管理 |
| S-07 | P3 | 协议无认证、无加密 | 全程 | 内网可用；外网需鉴权 |
| S-08 | P3 | `payloadCrc32` 恒为 0 | 所有发送路径 | 可考虑启用校验 |
| S-09 | P3 | `(SOCKET)sock_` 与 `static_cast<SOCKET>` 混用 | `MotorPlc.cpp` | 统一风格 |
| S-10 | P3 | `recv` 返回 0（对端关闭）与 <0（错误）未区分 | `MotorPlc.cpp:251` | 分别记日志，便于现场排查 |
| S-11 | P2 | `QCameraProperyTreeView` 空指针 | `QtDemo/QCameraProperyTreeView.cpp:73-78` | `pData` 为空直接解引用（作者已标 TODO） |
| S-12 | P2 | `timestampUs` 用 `steady_clock`（开机以来微秒） | `GmkServer` 组帧处 | 下游可能误当墙钟时间；补注释或改 `system_clock` |

## 11.6 PLC 封装（PDUMotionWrapper）

| 编号 | 级别 | 问题 | 位置 |
|---|---|---|---|
| P-01 | P1 | 析构为空，`m_p` 未释放 → 内存泄漏且 TCP 线程不停 | `PDUMotionWrapper.cpp:21` |
| P-02 | P1 | `_WaitForRespond` 重复发送同一命令（调用方已 `send` 过一次） | `PDUMotionWrapperPrivate.cpp:375` |
| P-03 | P2 | `enumcard` 解析丢弃最后一个元素（无尾逗号时未收尾） | 同上 `:343` |
| P-04 | P2 | 无法识别的报文被当作 `enumcard` 解析 | 同上 `:328` |
| P-05 | P2 | 各业务函数不检查连接状态 | 同上 |

## 11.7 构建与工程

| 编号 | 级别 | 问题 | 建议 |
|---|---|---|---|
| T-01 | P2 | `.gitignore` 第 23 行 `*.json` **误伤参数白名单配置** | 加 `!src/**/*.json` |
| T-02 | P2 | `.sln` 有 `MSB5023` 嵌套节错误（GMKdemo GUID 未注册） | 修 `.sln` 或迁 `.slnx` |
| T-03 | P3 | `Core.vcxproj` PostBuildEvent 曾被清空 | 已用 `git checkout` 恢复 |

## 11.8 文档与测试

| 编号 | 级别 | 问题 | 影响 |
|---|---|---|---|
| Q-01 | P1 | `GMKWin.md` 承诺 17 个测试文件（1,209 项检查），**`src/GMKdemo/test/` 不存在**（已核实） | 无回归保护，文档结论无法复现 |
| Q-02 | P2 | 文档说 `async` 支持左右独立张数，代码已强制共用 | **以代码为准** |

> ⚑ 核对：`Test-Path src\GMKdemo\test` 返回 `False`，Q-01 成立。`GMKWin.md` 的第 19–35 行列出的测试文件全部不存在，属历史承诺。

## 11.9 本会话已处理项

**仓库内（7 项，`git status` 可见）**：

```
M .gitignore                            QtMSBuild 忽略规则
M include/LPopupMsg.h                   const 修复
M src/CameraManager/CameraManager.cpp   const 修复
M src/Core/CvHelper.cpp                 临时变量修复
M tpvs17/GMKdemo/GMKdemo.vcxproj        QtInstall 绝对路径
M tpvs17/QtDemo/QtDemo.vcxproj          QtInstall 绝对路径
M tpvs17/lpc.Net/lpc.Net.csproj         .NET 4.0 → 4.8
```

**仓库外（不影响 git）**：

```
D:\git_IBSG\salticidae.props                                   props 副本
%LOCALAPPDATA%\...\Microsoft.Cpp.x64.user.props                GMK 变量定义
D:\git_IBSG\Output\x64\Release\runner17\salticidae\            运行时产物
```

---

# 第 12 章 认知修正录

> 原稿两份（第 16 章 12 条、19.5 节 8 条），按主题合并为 19 条，另加 1 条澄清（12.4）。**这一章的价值高于其他章**——它记录的是"以为是这样，其实不是"。

## 12.1 分层与职责

| 初始理解 | 实际 |
|---|---|
| GMKdemo 依赖 `CameraManager` 类 | ❌ 只用 `L*` C 函数，门面层在 `lpcManager.dll` 内 |
| `Core` 只服务相机 | ❌ 72 个工程共用（含电源、运动控制） |
| 外部依赖只在 `GMKBoardWindow` 和 `IBFusionRunner` | ⚠️ 还有 OpenCV / nlohmann / Qt / GalaxyEnv |
| `LibraryHelper` 负责扫描文件夹 | ❌ 只负责加载调用，扫描在 `CameraLibraryHelper` |
| `CameraCtrl` 是驱动的基本单位 | ❌ 是**驱动包装层**，`ICameraObject` 才是契约 |
| `IBProcessingPipeline` 负责分支判断 | ❌ 分支在 `GMKBoard`，pipeline 只提供能力接口 |
| `IBProcessingPipeline` 有构造函数 | ❌ 全成员可默认构造，靠 `Load()` 显式初始化 |
| `IBServiceRuntime` 管理任务实例 | ❌ 只管引擎开关，任务由 Runner 管理 |

## 12.2 通信与协议

| 初始理解 | 实际 |
|---|---|
| 协议发送 `<RUNSTEP,1,1;>` | ❌ 实际**不带尖括号**，只在接收时容错 |
| `TCP V3` 帧格式是项目自定义 | ❌ 是外部规范 `TCP_V3.md`；项目只改了 CRC、上限、双通道、消息号 |
| 处理完的图像传给 **PLC** | ❌ 传给**上位机**；PLC 只收文本命令，不收图 |
| 上位机会下发图片 | ❌ 只下发命令和参数（离线模式让板子读本地磁盘） |
| `ListenerLoop` 里 `channel` 会变化 | ❌ 是线程启动时的**固定参数**，全文不变 |

## 12.3 并发与时序

| 初始理解 | 实际 |
|---|---|
| `StartCapture` 内部调用 `PrepareTaskForCapture` | ❌ 反过来：`PrepareTaskForCapture` → `StartCapture`，两者在 `HandleControl` 里**同步串行** |
| `captureWorker_` 和 detach 线程是重复设计 | ❌ **正交**：前者保护 UI，后者保护 `ControlLoop` |
| 连续模式板端 `for` 循环拍 14 次 | ❌ 循环在 **PLC 内部**，板端只响应 14 次 `"3"` |
| 4 个线程对应 4 种任务类型 | ❌ 对应**一张图拆成的 4 个区域** |
| 拆图→融合→评分是串行流程 | ❌ 三种模式**互斥**，评分与融合不共存 |
| 锁内归还会死锁 | ⚠️ **不会**（无反向路径），但会阻塞相机回调并放大等待 |

## 12.4 相机到底受谁控制

第 1 章说"相机不是 PLC 控制的"，第 8 章又说"拍照由 PLC 的 `3` 驱动"——两句话都对，说的是不同层面：

| 说法 | 含义 |
|---|---|
| 相机不受 PLC 控制 | **控制权**在板端：相机 SDK 调用、参数、启停都由板端发起 |
| 拍照由 `3` 驱动 | **时机**由 PLC 决定：PLC 走到位才发信号，板端据此触发 |

一句话：**PLC 提供节拍，板端执行动作。**

---

# 第 13 章 结论与下一步

## 13.1 五条结论

1. **板端是"时序协调器"**，不是算法中心：运动节奏交给 PLC，融合交给 IB，它只负责把三者串起来并在正确的时刻发图。
2. **相机库的复杂度在属性抽象与插件兼容**，不在取图本身；读代码应先看 `CameraCtrl` 与 `PropertyPrivate`，而不是某个驱动。
3. **性能瓶颈在内存带宽与拷贝**，不在 CPU：98.8 MiB/帧、8ms 间隔，决定了缓冲池复用是硬需求。
4. **正确性瓶颈在"完成"的定义**：`TASK_DONE` 要等队列空 + `inFlight == 0` + 电机回位，三者缺一就会出现"客户端 0 张图"。
5. **当前最大风险是环境不可自举**：IB 包缺失、测试目录缺失、props 依赖本机路径——换机即重来。

## 13.2 待确认事项

| # | 待确认 | 影响范围 |
|---|---|---|
| 1 | **双相机真实业务场景**（拼接覆盖 vs 独立区域） | 决定后续所有双相机开发方向 |
| 2 | `INTERVAL` 的 `leftStartStep` / `rightStartStep` 在 PLC 侧如何使用 | 双相机异步调试 |
| 3 | 是否要恢复"左右独立张数" | 需现场验证 |
| 4 | 拆图固定 4 块的原因（算法限制？精度？内存？） | 改块数的风险评估 |
| 5 | `plugins\` 驱动加载依赖 PATH 而非显式扫描，是否改为 exe 同目录 | 驱动部署可靠性 |

## 13.3 下一步

| 优先级 | 事项 | 理由 |
|---|---|---|
| **P0** | 解决 IB 部署包问题（E-01） | 阻塞主链路，其他都无从验证 |
| **P1** | 确认双相机业务场景（待确认 #1） | 决定开发方向，避免白做 |
| **P2** | 修 S-01 / S-02 / S-03（服务器三个 P1） | 现场表现为"重连后收不到图""大图静默丢" |
| **P3** | 补回测试骨架（Q-01） | 当前零回归保护 |
| **P4** | 统一 UTF-8（B-08） | 影响调试体验，成本低 |

---

# 附录

# 附录 A 概念速查

> 原稿第 8、12、18 章三张速查表合并、去重，按领域分组。

## A.1 网络与 TCP

| 概念 | 解释 |
|---|---|
| `WSAEWOULDBLOCK` | "操作进行中/暂未就绪"，**不是错误**；非阻塞 `connect` 必返回它 |
| `FIONBIO` | `ioctlsocket` 的命令，切换阻塞/非阻塞模式 |
| `FIONREAD` | 查询接收缓冲区可读字节数，用于非阻塞探测 |
| `SO_ERROR` | 连接是否真正成功的最终判据 |
| 懒连接 | 首次发命令时才 `connect`，失败不缓存 |
| 长连接 | 一条 socket 全程复用，仅异常或主动重连时断开 |
| `SO_SNDBUF` / `TCP_NODELAY` | 发送缓冲 8 MiB / 关 Nagle，缺一个吞吐就掉 |

## A.2 相机库

| 概念 | 解释 |
|---|---|
| `ICameraObject` | 插件契约，8 个纯虚函数，**不含同步与属性操作** |
| `CameraCtrl` | 单相机代理，**持有**（非继承）`ICameraObject`，状态机/缓存/拆图在此 |
| `CameraNewInstance` | 插件工厂导出函数，**运行时字符串查找**而非编译期接口 |
| `RegisterCamera` | 驱动注册属性 + 框架补公共属性（`SplitByRow` / `BayerMode`） |
| PUSH / PULL | 循环所有权：SDK 跑循环 / 插件跑循环 |
| `GroupFrame` | 按 SN 分槽的表，**判齐靠 `bValid` 而非时间戳** |
| `External` 触发 | **唯一真硬件同步**路径 |
| `SplitByRow` / `AppendLastRows` | 拆图 / 拼图，方向相反 |
| 公共属性 | 所有相机都有、由框架实现，驱动不用重复写 |

## A.3 GMK 业务

| 概念 | 解释 |
|---|---|
| `BoardParams` | 界面层参数（`std::wstring`），`SET_PARAMETERS` 直接写入 |
| `CaptureRequest` | 电机请求（内部中转），含 `travelMm` / `moveIntervalMs` |
| `CaptureTask` | 抓拍任务（运行时状态），含计数器与完成标志 |
| `ProcessingOptions` | 融合参数快照（每组抓拍冻结） |
| `ProcessingSettings` | 图像输出参数：`scale` / `targetWidth` / `targetHeight` / `cropPosition` / `previewFps` |
| `FrameJob` | 处理队列单元，`jobId` 空 = 预览帧，非空 = 抓拍帧 |
| `FusionInputFrame` | 送算法的输入，**时间戳只透传不改** |
| `FusionResult` | 算法输出：`splitImages[4]` + `topLefts[4]` + `realtimeImage` + `depthFusionImage` + `needStop` + `clarity` |
| `fusionOutputDiagnostic` | 融合图缺失时的**取证字符串**（task id + 原因 + 实际输出清单） |
| `motionRunning_` | PLC 还在动，`MaybeFinishTask` 必须等它清零才发 `TASK_DONE` |
| `EndMotionRun()` | 唯一允许发 `TASK_DONE` 的位置 |
| `plcSequenceMutex_` | 串行化整个抓拍的 PLC 交互，期间手工指令被拒 |
| `softTriggerRejectStreak_` | 连续被拒 5 次后本任务内不再调 `LSnapCamera`（外触发自适应） |
| `kMaxPooledInputBuffers` | 每通道缓冲池上限 3（每帧约 114 MB） |
| `AcceptPreviewFrame` | 预览限帧，**按通道独立计时** |
| `previewVersion_` | 预览参数版本号，避免每帧重复 `Configure` |
| `TimestampSource` | `steady_clock`（开机以来微秒），**非墙钟时间** |

## A.4 板间通信

| 概念 | 解释 |
|---|---|
| `streaming_` | "状态流独占"标志，拒绝抢占而非阻塞 |
| `inFlight` | 已出队但仍在写的帧，`WaitChannelDrained` 必须等它归零 |
| `TryLockPlcSequence` | 手动命令抢抓拍锁，抢不到就放弃（不排队） |
| `SendOnly` | 流运行期间的"插入"通道，保证急停必达 |
| `WaitChannelDrained` | 发 `TASK_DONE` 前的排空等待（队列空 + `inFlight == 0`） |

## A.5 构建

| 概念 | 解释 |
|---|---|
| `$(SolutionName)` | 为空时输出路径少一层（命令行单编的坑） |
| `$(MSBuildThisFileDirectory)` | props 位置决定 `Output` 落点 |
| Delay Import | `lpvIB → lpbengine` 延迟加载，**启动时不检查** |
| `0xC0000135` | `STATUS_DLL_NOT_FOUND`，通常是 DLL 不在 exe 同级目录 |
| `MSB803` | 缺指定版本 Windows SDK |
| `LNK1181` | 找不到 `.lib`，通常是 props 位置错 |

---

# 附录 B 配置与部署速查

## B.1 GMKdemo.ini 全键

| 段 | 键 | 含义 |
|---|---|---|
| `[Server]` | `bind_ip` | 监听地址，默认 `0.0.0.0` |
| | `control_port` / `left_port` / `right_port` | 9500 / 9501 / 9502 |
| `[Plc]` | `host` / `port` | `192.168.10.200` / 6000（EL）、6001（MMC） |
| | `timeout_ms` | 命令超时，默认 5000 |
| `[Camera]` | `device` | 驱动名，如 `DaHeng (V2.6.2608)` |
| | `galaxy_sdk` | GalaxySDK 目录（相对 exe 或绝对） |
| | `capture_timeout_ms` | 取图超时 |
| `[Fusion]` | `ib_path` | IB 目录 |
| | `task_path` | 融合 task 文件 |

## B.2 MSBuild 变量

| 变量 | 用途 | 缺失后果 |
|---|---|---|
| `GMKLPVBin` | lpv SDK 的 `bin\x64` | `CopyLpvRuntime` 失败 |
| `GMKIBDir` | IB 目录 | `MSB3030 找不到 lpbengine.dll` |
| `GMKGalaxySdk` | GalaxySDK 目录 | 大恒驱动加载失败 |

**技巧**：`GMKIBDir` 指向不存在目录 → Copy 跳过 → 避免 MSB3030。

## B.3 运行时目录

```
Output\x64\Release\runner17\salticidae\
├─ GMKdemo.exe
├─ lpcManager.dll  Core.dll  lpvCore.dll  lpvGeom.dll  lpvIB.dll
├─ lpbengine.dll
├─ GMKdemo.ini
├─ plugins\         lpc*.dll + GxIAPICPPEx.dll 等
├─ GalaxySDK\       APIDll / GenICam / GenTL
└─ data\            salticidae\config\log4cplus.properties
```

## B.4 环境变量

| 变量 | 谁设 | 说明 |
|---|---|---|
| `PATH` | LPC 自动 | 追加 `plugins\` 下每个子目录（只增不减） |
| `GENICAM_GENTL64_PATH` | GMKdemo 自动 | 指向 `GalaxySDK` 的 GenTL 目录 |

---

# 附录 C 重叠审计与原稿对照表

> 这一节回答"笔记里哪些地方在重复"，也是本版的施工图。

## C.1 重复内容清单（8 组）

| # | 主题 | 原稿位置 | 重复的具体内容 | 本版去向 |
|---|---|---|---|---|
| 1 | **PLC 链路** | 2.1–2.8 与 19.1 | 端点、ASCII 协议、懒连接、五种收发、线程模型各讲两遍；19.1 的"5 种形态表"与 2.6 的"5 个入口表"是同一件事的两种切法 | 第 8 章（两表合并为 8.7） |
| 2 | **上位机链路** | 3.1–3.6 与 19.2 | 三端口、24 字节帧头、线程模型各两遍；3.5 的"发送队列"与 19.2 的"跨口超车"讲同一机制 | 第 9 章 |
| 3 | **并发与互斥** | 第 4 章、19.3、2.7、3.4 | 拒绝矩阵、优先级、`streaming_` 在三处出现 | 第 10 章 |
| 4 | **速查表** | 第 8 章、第 12 章、第 18 章 | `ICameraObject`、`CameraCtrl`、PUSH/PULL、`GroupFrame`、`0xC0000135`、`$(SolutionName)` 在 12 与 18 重复 | 附录 A（去重后 5 组） |
| 5 | **问题清单** | 第 7、11、14、19.6 章 | 四份共 40+ 条，`_AddDirectoriesToPath`、`LPC.h` 大小写、日志乱码各出现两次 | 第 11 章（统一编号 E/F/D/B/S/P/T/Q） |
| 6 | **构建问题** | 第 10 章、14.1、11.3 | 9 条报错、props 位置、DLL 搜索在两处讲 | 第 2 章（正文）+ 11.1/11.7（台账） |
| 7 | **认知修正** | 第 16 章、19.5 | 两份并列，主题重叠（如"StartCapture 与 Prepare 顺序"） | 第 12 章（20 条并 4 组） |
| 8 | **架构与职责** | 第 1 章、9.1–9.2、13.1–13.2 | GMKdemo 模块分工讲了两遍；"三层抽象"与"装配链"分散 | 第 1、3、7 章 |

## C.2 局部重复（同章内）

| 位置 | 重复内容 | 处理 |
|---|---|---|
| 2.4 与 2.5 | 懒连接 → 非阻塞 connect 的因果讲了两遍 | 合成 8.5 → 8.6 的递进 |
| 9.5 与 9.6 | 都提到"融合/多相机"，但对象不同 | 单列 5.6 做对照表 |
| 13.3 与 19.4 | 数据流讲了两遍，19.4 只多了七步优化 | 合成 7.4 + 7.5 |
| 3.2 与 3.4 | 端口表与线程图信息重叠 | 保留，前者讲职责后者讲生命周期 |
| 10.4 与 11.9 | 验证输出与改动清单都列了产物 | 验证进 2.6，改动进 11.9 |

## C.3 说法冲突（已裁定）

| 冲突点 | 原稿 A | 原稿 B | 本书裁定 |
|---|---|---|---|
| 相机是否受 PLC 控制 | 第 1 章："相机不是 PLC 控制的" | 19.1："拍照由 PLC 的 3 驱动" | 都对，分层表述（12.4）：**PLC 给节拍，板端执行** |
| 提交数与版本 | 9.1：1,629 次、HEAD 1.5.6 | 当前 git：1,600 次、`1.4.29-50` | 以当前为准，标注分支差异 |
| GMKdemo 规模 | 13.2：39 文件 / 8,001 行 | 实测：35 文件 / 11,698 行 | 以实测为准（7.1），注明口径含 `.ui` |
| 测试目录 | 14.4：`test/` 不存在 | `GMKWin.md`：列了 17 个测试文件 | **`test/` 确实不存在**（已 `Test-Path` 验证），文档过期（Q-01） |
| `async` 左右独立张数 | `GMKWin.md`：支持 | 代码：强制共用并告警 | 以代码为准（6.6） |
| 9502 端口 | 3.2：单相机方案可关闭 | 19.2：不监听 | 同一件事的两种说法，统一为"不监听" |

## C.4 原稿章 → 本书章

| 原稿 | 本书 |
|---|---|
| 1 总体架构 | 1.2、6.3 |
| 2 PLC 链路 | 8（+ 19.1） |
| 3 上位机链路 | 9（+ 19.2） |
| 4 并发与互斥 | 10.2、10.3 |
| 5 EL / MMC | 8.10 |
| 6 上位机客户端 | 9.9、9.10、1.4 |
| 7 问题与建议 | 11.5、11.6 |
| 8 概念速查 | 附录 A.1 |
| 9 LPC 架构 | 3、5 |
| 10 构建实录 | 2 |
| 11 代码审查 | 11.2、11.3、11.7 |
| 12 概念速查（LPC） | 附录 A.2 |
| 13 本周完成 | 7.1–7.7 |
| 14 问题清单 | 11.1、11.4、11.8 |
| 15 待确认 | 13.2 |
| 16 认知修正 | 12 |
| 17 下一步 | 13.3 |
| 18 速查补充 | 附录 A.3 |
| 19 深层走读 | 8（19.1）、9（19.2）、10（19.3）、7.5（19.4）、12（19.5）、11（19.6） |

## C.5 被删掉的内容

| 内容 | 原因 |
|---|---|
| "本周完成""本会话改动清单"的时间线叙述 | 属工作日志，已压缩为 11.9 |
| 重复的"易错点"小结 | 与正文表格重复 |
| `GMKWin.md` 已覆盖的工程细节（ini 全键、构建步骤） | 保留交叉引用，本书只留决策所需 |
| 原稿第 19 章末尾的重复问题表 | 已并入第 11 章 |

---

# 附录 D 复习自测 25 题

> 合上书答，答不上来的回对应章节。答案在题后括注。

**架构**

1. `CameraCtrl` 和 `ICameraObject` 是什么关系？（持有，非继承 → 3.2）
2. 一个 `lpc*.dll` 至少要导出哪几个函数？（6 个 → 3.4）
3. `LAddCamera` 内部为什么可能很慢？（会临时开相机读参数 → 3.3）
4. PUSH 与 PULL 怎么快速区分？（搜 `std::thread` / 看 `IStartCamera` 末参 → 3.5）
5. 为什么 Artray 把 SDK 头藏在 `.cpp`？（不污染上层 `.h` → 3.6）

**功能**

6. 未 `Open` 直接 `Start` 会怎样？（报 `CameraNotOpen` → 4.2）
7. 内触发模式下调 `Snap` 返回什么？（`TriggerModeError` → 4.2）
8. 属性三元信息分别解决什么问题？（易变 / 需停相机 / 联动刷新 → 4.3）
9. `SplitByRow` 与 `AppendLastRows` 的区别？（拆 vs 拼 → 5.5）
10. 组帧"齐了"的判据是什么？（每个 SN 的 `bValid` → 5.2）
11. 哪种触发模式是真同步？（`External` → 5.3）

**GMK 业务**

12. 相机是谁控制的？（板端直接控制，PLC 只给节拍 → 1.2、12.4）
13. 连续模式下 14 次循环在哪执行？（PLC 内部 → 8.4）
14. 为什么 `StartCapture` 必须 detach？（否则控制通道卡死，收不到 CANCEL → 10.4）
15. 为什么 4 个融合实例？（一张图拆 4 块，实例内有累积状态 → 7.7）
16. 停拍评分和融合能同时跑吗？（不能，`if/else` 互斥 → 7.7）
17. 一帧进入算法前做了哪七步？（7.5）
18. 左右通道为什么各用一把锁？（共用会让一路吃掉全部资源 → 7.8）

**通信**

19. 请求报文带尖括号吗？（不带，接收容错 → 8.2）
20. 为什么必须非阻塞 `connect`？（默认阻塞约 21 秒 → 8.6）
21. 为什么 PLC 要五种收发模式？（防止 `CHK` 抢走状态流的 `3`/`4` → 8.7）
22. `inFlight` 解决什么？（帧已出队但仍在 `send`，误判排空 → 9.7）
23. 为什么 `kMaxPayload` 从 64 MiB 改 256 MiB？（单帧 98.8 MiB 装不下 → 9.2）

**构建**

24. `$(SolutionName)` 为空会怎样？（输出少一层目录 → 2.2）
25. `0xC0000135` 最常见的原因？（DLL 不在 exe 同级 → 2.7）

---

**（全书完）**

> 维护建议：修改代码后，同步更新第 11 章的编号状态（修好的标 ✅）与第 2.6 的验证输出；新增驱动按 3.4 的六函数清单核对。


