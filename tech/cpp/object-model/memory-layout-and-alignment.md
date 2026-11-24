---
id: cpp/object-model/memory-layout-and-alignment
title: 类对象布局与内存对齐
level: null
level_status: 待用户确认
reviewed: 2026-11-01
next_review: 2026-11-03
projects: []
sources: [本轮聊天整理]
---

## 一句话

成员按自身对齐要求放置，整体大小补到类型对齐要求的整数倍；大小不等于对齐要求。

## 机制
- 以下限于无特殊设置的普通布局题：整体对齐通常取成员最大 alignof；每个成员起始偏移满足自身 alignof，最后按整体对齐补尾部。真实规则受 ABI、继承、alignas、打包设置等影响。
- sizeof 包含填充，alignof 查询对齐。对齐满足硬件和平台访问要求，非对齐访问可能额外耗时或触发异常；不满足语言要求的类型访问可能是 UB。
- 假设 char 大小/对齐 1、short 为 2、int 为 4、double 为 8：
  - char,char,int：偏移 0,1,4，大小 8，对齐 4。
  - char,int,short：偏移 0,4,8，尾补 2，大小 12。
  - char,double,int,char：偏移 0,8,16,20，尾补 3，大小 24，对齐 8。
- Inner{char,int} 大小 8、对齐 4；Outer{char,Inner,char} 偏移 0,4,12，大小 16。不能因 Inner 占 8 就按 8 对齐。
- 非静态数据成员及填充占对象空间；静态数据成员不在每个对象内，普通成员函数代码不按对象复制。
- 空类完整对象 sizeof 至少 1，通常 1；空基类子对象可能通过 EBO 不额外占空间，不能简单相加基类 sizeof。普通空成员通常占空间，特殊属性另论。
- class 与 struct 没有由关键字直接造成的额外布局开销。

## 追问链
1. char 后为什么填 3？→ 为紧随其后的 4 对齐成员；不是 char 自己要求。
2. 尾部填充为什么需要？→ 保证数组下一个对象及其成员仍对齐。
3. 偏移与大小？→ 前者是起点距离，后者是占用字节数。

## 项目证据 / 反例
- 用户算对前三题；嵌套题曾把 sizeof=8 当 alignof=8，重点复习。

## 关联
- [[cpp/object-model/encapsulation-and-inheritance]]
- [[cpp/object-model/virtual-dispatch]]
