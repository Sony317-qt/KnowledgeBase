---
id: projects/salticidae/incidents/2026-11-20-async-left-right-count
date: 2026-11-20
type: 文档与实现不一致（非线上事故）
---

# 2026-11-20 · 文档说 async 支持左右独立张数，代码已强制共用

## 症状

`GMKWin.md` 与请求字段说明都写着：`motion_mode=async` 时可以带
`left_start_step / left_count / right_start_step / right_count`，左右各自起始步与张数。
按文档实现客户端功能，发现右边张数永远等于左边。

## 定位

翻 `GMKBoard` 的抓拍请求处理路径：`motionMode=async` 与 `continuous` 实际走**同一路径**，
`right_count` 被强制赋值为 `left_count` 并打印警告。

## 根因

文档落后于实现。原笔记的结论是：

> ⚑ 文档与实现不一致，**以代码为准**：`GMKWin.md` 说 `async` 支持左右独立张数，
> 代码已强制 `right_count = left_count` 并打警告。

## 影响

- Q-02 记录在质量问题台账（文档与测试类）。
- 待确认事项 #3："是否要恢复左右独立张数" —— 需现场验证。
- 更根本的问题是：**协议字段还在，但语义已被单方面废掉**，客户端无从得知。

## 修法 / 教训

- 契约类字段（协议语义）被收紧时，必须同步改文档**或**升版本、加能力位；
  "打一条警告日志"不足以通知远端客户端。
- 走读代码时，判断 `motionMode=async` 要**同时看请求端与执行端**，只读一边必然得出错误结论。

## 关联

- [[projects/salticidae/README]] —— 抓拍模式与 Q-02
- 同类问题：[[incidents/2026-11-20-plc-message-syntax]]（文档描述的报文语法也与实现不同）
