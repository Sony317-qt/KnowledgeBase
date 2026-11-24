# BorderCollie 项目学习笔记（四条主线）

> 整理自本次会话的代码阅读与问答，覆盖四条主线：**IB 服务加载 → 相机与数据源 → 通信 → 标定**。
> 行号以当前工作区代码为准（文件变动后可能偏移，以函数名为锚更稳）。
> 更全面的项目总览见 `diagram/00_overview.md`，模块细节见 `diagram/modules/*.md`。

```text
PLC 触发（通信入站）
   → 相机按“数据源”参数拍照（相机/数据源）
   → 图像进算法线程：LPV / IB 任务执行（IB 服务）
   → 标定把像素坐标转成物理坐标（标定）
   → 结果按协议写回 PLC（通信出站）
（拍照前后由光源模块按绑定表点亮/熄灭对应通道）
```

---

## 1. 第一条：IB 服务是怎么加载进来的

### 1.1 一句话

主窗口启动时创建**唯一**的 `IBservice`（`LIBService::Create()` + `SetUp(ib_path)`），再把同一个 `ILIBServicePtr` 注入算法工厂、信号站、标定和内置定位器；`AlgIB` 拿到它之后用 `LoadTask / ExecuteTaskSync` 执行灵闪（IB）流程。

### 1.2 加载时序

```text
main.cpp（不创建 IB）
 └─ MainWindow 构造
      ├─ ib_service_ = new IBservice()                      mainwindow.cpp:186
      │    ├─ IBservice::IBservice()                        ibservice.cpp:7
      │    │    ├─ LIBService::Create()
      │    │    └─ 读 file/ib.json → "ib_path"
      │    └─ loadIB(path)                                  ibservice.cpp:107
      │         ├─ ibService_->SetUp(path)                  ibservice.cpp:114
      │         └─ LoadGlobalAction(task/globalAction.json) ibservice.cpp:120
      ├─ industry_->setIB(ib_service_->getIB())             mainwindow.cpp:193
      ├─ signal_station->setIB(ib_service_->getIB())        mainwindow.cpp:194
      ├─ calib_->setIB(ib_service_->getIB())                mainwindow.cpp:374
      └─ 切换行业 DLL 后重新 setIB                          mainwindow.cpp:2693
```

注意：**IB 不是 main.cpp 里初始化的**，而是主窗口装配阶段；`file/ib.json` 的 `ib_path` 为空时不会加载。

### 1.3 注入给谁、谁在用

| 消费方 | 用途 | 位置 |
|---|---|---|
| `industry_`（DLL 工厂） | 只存不用的“中转站”，`setIB/getIB` 原样返回 | `industry/Connector/AlgLpvConnectorInterface.cpp` |
| `WidgetAlg::locator_`（内置定位器） | 从 `industry_` 取回 IB，传给 `AlgIB` | `Modules/WidgetAlg/widgetAlg.cpp:747`、`:987`、`:1353`、`:1852` |
| `AlgIB` | 构造时 `LoadTask`；运行时 `ExecuteTaskSync`；卸载 `UnloadTask` | `Modules/Industry/locator/AlgLpvLocator.cpp:744`、`:759`、`:762`、`:876`、`:994` |
| `SignalStation` | 通信参数读写 IB 全局变量 | `Modules/CommonTool/SignalStation/signalstation.cpp:703-730` |
| `DialogCalib` | 多点标定里跑 IB 任务 | `Modules/DialogCalib/dialogcalibnpoints/` |

### 1.4 为什么只建一个实例

- `SetUp()` 要加载整个 IB 工程（任务 + 全局变量 + 算子库），开销大；
- IB 全局变量是**进程级共享**，多实例会互相冲突（代码里有 `globalVarConflict` 处理）；
- 生命周期归主程序（`IBservice` 成员），通过 `ILIBServicePtr`（引用计数）分发，DLL 卸载也不影响。

### 1.5 易错点

1. `ib_path` 为空 / `SetUp` 失败 → `IsReady()==false` → `AlgIB` 创建失败，界面上加不出 IB 算子。
2. 注入必须早于算子创建：`AlgIB` 构造即 `LoadTask`。
3. `task/globalAction.json` 加载失败只记 `qCritical`，不拦截启动。
4. 线程：IB 调用发生在算法线程（`widgetAlgThread`），不在 GUI 线程。

---

## 2. 第二条：相机模块与数据源

### 2.1 三层结构

```text
DialogCamera              UI + 配置：相机列表、数据源页、camera_config.json
  └─ DialogCameraSalticidae   管理：枚举/添加/删除/切源/参数下发/软触发/启停流
       └─ CameraSalticidaeThread   每相机一个线程：调 LPC、注册回调、LFrame → CameraImage
```

### 2.2 图像回调链（SDK 线程 → Qt 线程）

```cpp
// dialogCameraThread.cpp
LRegisterCameraImageCallBack(..., this, &register_id)      :465
LRegisterEventNotifyCallBack(LEventStateEvent | LEventCamera, ...)  :478
```

- LPC 是 C API：回调带 `void* pUser`，里面 `static_cast` 回 `CameraSalticidaeThread`；
- 回调把 `LFrame` 转成 `cv::Mat` / `image_3d_source`，封装为 `CameraImage`；
- 用 `emit snapFinish(img)` 从 SDK 线程弹回 Qt 线程（跨线程自动排队）；
- 注销必须早于 `LCloseCamera/LDeleteCamera`，否则 `pUser` 悬空。

### 2.3 数据源是什么

**定义**：一台相机在某种拍摄条件下的一套预设方案（`class DataSource`，`dialogCameraDefine.h:144`），不是硬件、不是一帧图、不是运行时状态。

| 项 | 结论 |
|---|---|
| 内容 | 名字、软触发延时、2D 参数、2D 线扫参数、3D 参数、基恩士 3D 参数、虚拟相机路径 |
| 标识 | `(相机名, 源名)`；内部 `data_src_id` **0 基**，通信地址 `CCDx-y` **1 基** |
| 独占 | 算法目录 `model/<型号>/<相机>/<源>/`、管控、保存路径、光源绑定、绑码、通信地址、总览格子 |
| 业务单元 | 每个数据源对应一个 `WidgetUnit`（`WidgetUnit = 相机 + 数据源`） |
| 约束 | 不能跨相机共享；随动补偿、图像融合都限同一相机 |

### 2.4 切源流程

```text
PLC/ TCP 命令 SwitchDataSource
  → DialogCameraSalticidae::onSignalSwitchDataSource          dialogCameraSalticidae.cpp:470
  → switchDataSource(camera_id, data_src_id)                  :364
       ├─ 把目标源参数拷贝到当前相机并下发 SDK
       ├─ 期间断开 snapFinish，防旧帧进算法
       └─ 2D 线扫：额外重载配置文件
  → 软触发拍照（onSignalSoftTrigger，:530）
```

### 2.5 持久化与加载

| 动作 | 时机 |
|---|---|
| `cameraSaveJson`（dialogCamera.cpp:2741） | 关闭相机窗口（`on_cameraSignalfinished`，mainwindow.cpp:3658 → :3661） |
| `cameraLoadJson`（dialogCamera.cpp:2760） | 换型 / 启动加载型号 |
| 重命名数据源 | **例外**：立即保存 |

风险：改完参数没关窗口就崩溃 → 丢配置；目录同步会删除“不在 srcList 里的目录”。

### 2.6 易错点

1. 3D 数据源只有在**桌面 OpenGL 后端**下才会创建 `Widget3d`（`widgetUnit.cpp:295` 的 `AA_UseDesktopOpenGL` 判断）。
2. 数据源参数是值语义（`CameraPro::data_src` 是 `vector<DataSource>`），不要指望跨相机引用。
3. 切源期间不要触发生产，否则会拿到旧源参数拍出的图。

---

## 3. 第三条：通信模块

### 3.1 三个通道、两种结果形态

| 通道 | 类 | 结果形态 | 底层 |
|---|---|---|---|
| 文本 TCP | `ComTcp` | 字符串 `data_string` | TCP Server/Client |
| PLC 点对点 | `ComPointToPoint` | 数值 `p2p_data` + `p2p_result` | Modbus TCP / 基恩士 MC / 欧姆龙 FINS |
| IO | `ComIO` | 字符串是否等于 `OK`/`NG` | 研华 DAQNavi 位读写 |

`ComType`：0 TCP Server、1 TCP Client、2 Modbus TCP、3 基恩士 MC、4 欧姆龙 FINS TCP、5 IO。

### 3.2 建立连接与事件线程

```text
UI 连接按钮
 → DialogCommunication::pushConnected()                 dialogcommunication.cpp:287
 → ComPointToPoint::connectCom(ip, port)                communicationpointtopoint.cpp:69
      ├─ com_core_->connect()
      │     └─ ModbusCore: modbus_new_tcp + modbus_connect     commodbus.cpp:14
      ├─ registerEvent()                                communicationpointtopoint.cpp:746
      │     └─ 每个事件：new 对象 + 独立 QThread + QTimer(默认 200ms, 可配)
      └─ connected_ = true; emit connected()

断线：communicationDisconnected
 → reconnect_timer_.start(500ms)                        dialogcommunication.cpp:1788
 → reconnected() → reconnectCom() → 重新 registerEvent
```

### 3.3 入站与出站

| 方向 | 机制 | 关键点 |
|---|---|---|
| 入站 | 每个事件在自己的线程里 `QTimer` 轮询读地址 | 读到非 0 → emit → **写回 0**（握手复位）；读失败 → emit 断线 |
| 出站 | `ComResultSend::resultSend` | 先写 `p2p_data` 明细 → 条码 → **最后**写结果字 `1/2` |
| 命令回复 | `ComAssist::on_reply`（独立线程） | 写 `reply_address` |

入站例子（软触发）：`ComSoftTrigger::softTriggerListen`（communicationpointtopoint.cpp:1244）——读触发地址、非 0 触发、写 0；出站例子：`resultSend`（:988），`on_reply`（:2377）。

> 入站是**轮询**不是 PLC 推送，所以轮询周期直接决定响应延迟和 PLC 压力。

### 3.4 三种 P2P 协议与数据编码

| 协议 | 实现 | 地址模型 | 默认端口 |
|---|---|---|---|
| Modbus TCP | 原生 libmodbus | 纯数字寄存器（项目里 0 基） | 502 |
| 基恩士 MC | `PointToPointComInterface` + HslCommunication(.NET) | `D100` 这类软元件（项目只支持 D 区） | 库决定 |
| 欧姆龙 FINS TCP | 同上 | 内存区 + 地址（如 `D100`/`CIO100`） | 9600 |

**为什么三套**：帧格式、地址模型完全不同；现场 PLC 品牌不同；MC/FINS 是现成 .NET 库，Modbus 是原生手写。

**编码（P2P 层最小单位是 16 位字）**

| C++ 类型 | 占用 | 说明 |
|---|---|---|
| `bool` | 1 word | 写 0/1（Modbus 用 FC16 写寄存器，不是线圈） |
| `ushort` | 1 word | 原样 |
| `int` / `float` | 2 words | `memcpy`（x86 小端 → **低字在前**） |
| `string` | N words | Modbus：每寄存器高字节先；MC：每 word 内交换字节 |

字节序：**Modbus 协议本身是大端**（libmodbus 已处理 16 位），但 32 位值项目按小端内存布局；RTU 的 CRC 是小端。

### 3.5 易错点

1. 默认端口 8888 ≠ Modbus 502，第一次连不上先看这里。
2. 没有 `modbus_set_slave`，从站号用 libmodbus 默认（0xFF）。
3. Modbus 地址是十进制 0 基，不能填 40001。
4. 轮询周期 200ms，PLC 触发脉冲必须更宽；所有事件共用一把锁，事件多会串行。
5. 结果字最后写——PLC 应以结果字变化作为“明细已就绪”的信号。
6. 换 PLC 品牌不只是改下拉框：地址表、字节序假设、.NET 依赖（`HslCommunication.dll` 等）都要跟着改。

---

## 4. 第四条：标定模块

### 4.1 标定种类

| 类型 | 作用 | 实现 |
|---|---|---|
| 内参（FFD） | 畸变矫正 + 图像→世界 | `calibinternalthread.cpp:57-71` |
| 多点（9 点平移） | 像素↔物理平移映射 | `calibnpointsthread.cpp:132-142` |
| 联合（Joint） | 源相机→目标相机 + 机器人偏移 | `calibjointthread.cpp:83-89` |
| 随动（with） | 附加 offset（如机器人随动） | `CalibModel::with` |
| 示教（teach） | 对位示教偏移 | `CalibModel::teach` |

### 4.2 9 点平移流程

```text
画 ROI / 学模板（或加载 IB .task）
 → 机器人到位，下发坐标（CalibPos:CCD1,index,x,y,r）或手填
 → 触发拍照 → 模板匹配取中心
 → AddPoint(img, tar) × 9                       calibnpointsthread.cpp:132
 → Calibrate()                                  :134
 → ImageToWorld 反投影，算 RMS 物理误差          :142
```

通信索引：1–9 平移点，10–12 旋转点（`setPos` 里 `index--`）。

### 4.3 3 点旋转

- `LCalibRotation` + `predict()` 判断角度模型（`y=x+c` 或 `y=-x+c`）；
- 求旋转中心、角度补偿；
- 模式：`TranslationOnly / LocalRotation / DoublePointRotation`。

### 4.4 CalibModel 五层（`Modules/Industry/industryBase.h:20-165`）

| 层 | 作用 | 生效顺序 |
|---|---|---|
| `internal` | 畸变 + 图像→世界 | 最低 |
| `npoints` | 多点映射 + 角度修正 + 旋转中心 | 中 |
| `joint` | 源→目标 + `robot_offset` | 最高 |
| `with` | 随动 offset | `apply()` 时叠加 |
| `teach` | 示教 offset | `apply()` 时最后叠加 |

```cpp
applyCore(x, y, r)   // industryBase.h:151 —— 只用一层：joint > npoints > internal
apply(x, y, r)       // industryBase.h:158 —— applyCore 后依次叠加 with、teach
```

### 4.5 使用与持久化

- 持久化：`calib.json`（关闭标定窗口时写）；
- 运行时：`calib_model_map`（`Modules/Industry/locator`）供算法 `apply`；
- 标定拍照走 `calibSoftTrigger` 分支（不进生产算法链）；
- 算法里常见用法：`image_input_->calib.apply(x, y, r)` 把像素结果转物理坐标。

### 4.6 易错点

1. 配置了 `with_source_` 时 `getModel()` **只算 `with.offset` 就 return**，核心层为空 → `applyCore` 失败。
2. 联合标定统一用**终点**的标定矩阵。
3. `teach` 与 `with` 的叠加顺序不同（`apply` 里固定），改标定层时不要随意调换。
4. 标定数据是“型号级”的，换型会重新加载；标定没做完就跑生产会得到错误物理坐标。

---

## 5. 四条主线怎么串起来

```text
① PLC 触发（通信入站：轮询到触发字 → 写 0 复位）
      ↓
② 相机按当前数据源参数拍照（相机/数据源：软触发 → LPC 回调 → CameraImage）
      ↓
③ 图像进算法线程（IB 服务：AlgIB / LPV 算子执行）
      ├─ 标定：像素坐标 → 物理坐标（CalibModel::apply）
      └─ 结果组包（CommunicationData：字符串 + p2p_data + 结果字）
      ↓
④ 通信出站：先写明细、条码，最后写结果字 1/2 → PLC 读取并动作
```

四条主线各自的关键文件：

| 主线 | 入口文件 | 一句话 |
|---|---|---|
| IB | `Modules/Mainwindow/IBService/ibservice.cpp` | 建一个 IB，注入所有需要它的模块 |
| 相机/数据源 | `Modules/DialogCamera/` | 相机线程产图，数据源决定参数与方案 |
| 通信 | `Modules/DialogCommunication/` | 事件轮询入站、结果写寄存器出站 |
| 标定 | `Modules/DialogCalib/` + `CalibModel` | 把像素坐标变成物理坐标 |

---

## 6. 速查表

| 想找什么 | 看哪里 |
|---|---|
| IB 创建 / SetUp / 全局动作 | `ibservice.cpp:7`、`:107-121` |
| IB 注入 | `mainwindow.cpp:193`、`:194`、`:374`、`:2693`；`widgetAlg.cpp:747` 等 |
| IB 执行 | `AlgLpvLocator.cpp:744`（LoadTask）、`:876`（ExecuteTaskSync） |
| 相机回调注册 | `dialogCameraThread.cpp:465`、`:478` |
| 数据源结构 | `dialogCameraDefine.h:144` |
| 切源 | `dialogCameraSalticidae.cpp:364`、`:470` |
| 相机配置保存/加载 | `dialogCamera.cpp:2741`、`:2760` |
| P2P 连接/事件注册 | `communicationpointtopoint.cpp:69`、`:746` |
| 入站轮询 | `communicationpointtopoint.cpp:1244` |
| 出站结果 | `communicationpointtopoint.cpp:988`、`:2377` |
| Modbus 连接 | `commodbus.cpp:14` |
| 9 点标定 | `calibnpointsthread.cpp:132-142` |
| 内参标定 | `calibinternalthread.cpp:57-71` |
| 联合标定 | `calibjointthread.cpp:83-89` |
| CalibModel | `Modules/Industry/industryBase.h:20-165` |

---

## 7. 自测（能答出来就算这四条主线过关）

1. IB 服务是谁创建的？`file/ib.json` 里的 `ib_path` 为空会怎样？
2. 为什么 `IndustryBase` 要提供 `setIB/getIB`，而 DLL 自己又不用？
3. 相机回调为什么必须 `emit` 而不能直接调 UI？数据源的 `data_src_id` 是 0 基还是 1 基？
4. 切源时为什么要断开 `snapFinish`？
5. PLC 触发到拍照，中间经过哪几个对象？为什么软件读完触发字要写回 0？
6. 为什么结果字要最后写？`p2p_data` 里的 float 是两个寄存器怎么排？
7. 9 点标定的输入是什么、输出是什么？RMS 怎么算的？
8. `CalibModel::applyCore` 的优先级是什么？`with` 和 `teach` 在哪一步叠加？
