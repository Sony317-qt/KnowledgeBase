---
id: projects/bordercollie
title: BorderCollie —— 富士康定位软件（IB 服务 / 相机数据源 / 通信 / 标定）
role: 待补（学习走读 / 模块维护）
period: 2026
stack: [C++, Qt, IntelliBlink(IB), libmodbus, HslCommunication(.NET), 研华 DAQNavi]
status: 学习中（只读走读）
sources: [notes/bordercollie-study-notes.md]
---

## 一句话

BorderCollie 是一条**"PLC 触发 → 按数据源拍照 → IB/LPV 算法 → 标定转物理坐标 → 结果写回 PLC"**的
视觉定位软件，四条主线（IB 服务 / 相机与数据源 / 通信 / 标定）串成一条闭环。

## 30 秒陈述

> TODO(你补)：分清"我做的"与"我读懂的"。走读型项目不能包装成主导设计。

## 背景与约束

- **业务**：定位类检测——PLC 触发拍照，算法给出像素坐标，标定转成物理坐标，结果写回 PLC 驱动机构动作。
- **硬约束**：PLC 侧是**轮询**而非推送，轮询周期（默认 200 ms）直接决定响应延迟与 PLC 压力。
- **多品牌 PLC**：Modbus TCP / 基恩士 MC / 欧姆龙 FINS 三套协议并存。

## 技术栈与环境

| 项 | 内容 |
|---|---|
| 语言 | C++（主体）、C#（HslCommunication 下的 MC/FINS 位） |
| 框架/库 | Qt（信号槽跨线程）、IntelliBlink（IB / 灵闪）、LPV、libmodbus、HslCommunication(.NET)、研华 DAQNavi |
| 构建 | — （待补） |
| 平台 | Windows |
| 硬件 | 工业相机（经 LPC 相机库）、PLC（Modbus/MC/FINS）、研华 IO 卡、机器人 |

## 架构

```text
PLC 触发（通信入站）
   → 相机按"数据源"参数拍照（相机/数据源）
   → 图像进算法线程：LPV / IB 任务执行（IB 服务）
   → 标定把像素坐标转成物理坐标（标定）
   → 结果按协议写回 PLC（通信出站）
（拍照前后由光源模块按绑定表点亮/熄灭对应通道）
```

四条主线各自的入口：

| 主线 | 入口文件 | 一句话 |
|---|---|---|
| IB | `Modules/Mainwindow/IBService/ibservice.cpp` | 建一个 IB，注入所有需要它的模块 |
| 相机/数据源 | `Modules/DialogCamera/` | 相机线程产图，数据源决定参数与方案 |
| 通信 | `Modules/DialogCommunication/` | 事件轮询入站、结果写寄存器出站 |
| 标定 | `Modules/DialogCalib/` + `CalibModel` | 把像素坐标变成物理坐标 |

四条主线怎么串起来：

```text
① PLC 触发（通信入站：轮询到触发字 → 写 0 复位）
      ↓
② 相机按当前数据源参数拍照（软触发 → LPC 回调 → CameraImage）
      ↓
③ 图像进算法线程（AlgIB / LPV 算子执行）
      ├─ 标定：像素坐标 → 物理坐标（CalibModel::apply）
      └─ 结果组包（CommunicationData：字符串 + p2p_data + 结果字）
      ↓
④ 通信出站：先写明细、条码，最后写结果字 1/2 → PLC 读取并动作
```

## 关键决策

- [[decisions/0001-single-ib-instance]] —— IB 全局只建一个实例，靠注入分发
- [[decisions/0002-datasource-as-preset]] —— 数据源是"预设方案"，不是硬件也不是运行时状态
- [[decisions/0003-polling-inbound-address]] —— 入站靠轮询读地址 + 写回 0 握手
- [[decisions/0004-calib-layer-priority]] —— `joint > npoints > internal`，`with`/`teach` 最后叠加

## 事故 / 认知修正

- [[incidents/2026-11-20-calib-with-only-offset]] —— 配了 `with_source_` 时 `getModel()` 只算 offset 就返回
- [[incidents/2026-11-20-result-word-written-last]] —— 结果字必须最后写，PLC 靠它判断明细就绪
- [[incidents/2026-11-20-source-switch-old-frame]] —— 切源期间不防旧帧会拿到旧参数拍的图

## 我的职责与产出

- 按四条主线做代码走读，产出一份带行号锚点的学习笔记（原始笔记 325 行）。
- 梳理出每条主线的"一句话 / 时序 / 易错点"，并给出一份速查表与 8 道自测题。

> 走读型项目：不能对应到"我设计的技术点"，技术点最高标 2（见 README level 规则）。

## 量化

- 四条主线、8 道自测题。
- 通信：三种 P2P 协议、三个通道（TCP 文本 / PLC 点对点 / IO）、轮询默认 200 ms、断线重连 500 ms。
- 标定：5 种标定类型、9 点平移 + 3 点旋转、`CalibModel` 五层。

## 用到的技术点

- [[projects/bordercollie/notes/datasource-as-preset]] —— 数据源的语义
- [[projects/bordercollie/notes/p2p-word-ordering]] —— `memcpy` 小端映射到 16 位寄存器
- [[projects/bordercollie/notes/calib-model-five-layers]] —— 标定五层与叠加顺序
- [[cpp/concurrency/lock-scope-discipline]] —— 所有事件共用一把锁 → 事件多会串行

## 原始笔记（全文入库）

| 文件 | 大小 | 来源 |
|---|---|---|
| [[projects/bordercollie/source/bordercollie-study-notes]] | 15,758 B / 325 行 | `D:\BorderCollie\foxconnlocate\notes\bordercollie-study-notes.md` |

> 原文**未做任何改写**，逐字节复制（SHA256 `5432CF40…00FA`）。
> 目录下其他文件是从它抽取的索引/ADR/事故/技术点；**原文里的行号锚点、完整时序块、速查表与自测 8 题，都以原文为准。**
> 原文提到的 `diagram/00_overview.md`、`diagram/modules/*.md` 存在原仓库里，不在本库。

## 面包屑

- [[meta/index]]
