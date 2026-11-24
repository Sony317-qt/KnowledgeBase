---
id: projects/bordercollie/notes/p2p-word-ordering
title: P2P 通信的 16 位字布局与字节序
level: 2
tags: [communication, modbus, endianness, bordercollie]
aliases: [字节序, 字序, Modbus 编码, 寄存器打包]
reviewed: 2026-11-20
next_review: 2026-11-22
projects: [bordercollie]
sources: [notes/bordercollie-study-notes.md 3.4 节]
---

## 一句话

P2P 层最小单位是 **16 位字**：`int`/`float` 占 2 个字、`memcpy` 直接拍进去（x86 小端 → **低字在前**），
而 Modbus 协议本身是大端、RTU 的 CRC 是小端——**三个层次必须分开说**。

## 机制

| C++ 类型 | 占用 | 说明 |
|---|---|---|
| `bool` | 1 word | 写 0/1（Modbus 用 FC16 写寄存器，不是线圈） |
| `ushort` | 1 word | 原样 |
| `int` / `float` | 2 words | `memcpy`（x86 小端 → **低字在前**） |
| `string` | N words | Modbus：每寄存器高字节先；MC：每 word 内交换字节 |

三层字节序：

| 层次 | 是大端还是小端 | 谁处理 |
|---|---|---|
| Modbus 协议帧（含 16 位寄存器值） | 大端 | libmodbus 已处理 |
| 32 位值在寄存器里的排列 | **小端内存布局**（低字在前） | 项目自己 `memcpy` |
| RTU 模式的 CRC | 小端 | 协议栈 |

三套协议差异（为什么必须三套）：

| 协议 | 实现 | 地址模型 | 默认端口 |
|---|---|---|---|
| Modbus TCP | 原生 libmodbus | 纯数字寄存器（项目 0 基） | 502 |
| 基恩士 MC | `PointToPointComInterface` + HslCommunication(.NET) | 软元件（只支持 D 区） | 库决定 |
| 欧姆龙 FINS TCP | 同上 | 内存区 + 地址 | 9600 |

因为**帧格式、地址模型完全不同**，且现场 PLC 品牌不同：MC/FINS 是现成 .NET 库，Modbus 是原生手写。

## 追问链

1. **一个 float 怎么写进两个寄存器？**
   `memcpy(&regs, &f, 4)`，x86 小端 → 寄存器 0 = 低 16 位、寄存器 1 = 高 16 位。PLC 侧如果按大端字序读，就得交换。
2. **Modbus 不是大端吗？**
   是，但那是**协议层**对单个 16 位寄存器的编码；32 位值怎么拆成两个寄存器是**应用层约定**，项目选了小端。两者不矛盾。
3. **字符串怎么打包？**
   Modbus 每寄存器高字节先；MC 每 word 内交换字节。换 PLC 品牌时字符串最容易出错。
4. **换 PLC 品牌只改下拉框行不行？**
   不行：地址表、字节序假设、`.NET` 依赖（`HslCommunication.dll` 等）都要跟着改。

## 项目证据 / 反例

- **踩过的坑**：默认端口 8888 ≠ Modbus 502，第一次连不上先看这里。
- **踩过的坑**：没有 `modbus_set_slave`，从站号用 libmodbus 默认（`0xFF`）。
- **踩过的坑**：Modbus 地址是十进制 0 基，不能填 40001。
- **反例**："`bool` 用线圈（coil）写" ✗ —— 项目用 FC16 写寄存器，写 0/1。

## 关联

- [[projects/bordercollie/incidents/2026-11-20-result-word-written-last]] —— 写入顺序约定
- [[projects/bordercollie/decisions/0003-polling-inbound-address]]
