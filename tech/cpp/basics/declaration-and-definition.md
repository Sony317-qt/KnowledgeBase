---
id: cpp/basics/declaration-and-definition
title: 声明、定义与静态成员
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

声明介绍名字和类型；定义提供实体，定义也是声明。

## 机制
- 函数外的 `int a;` 是定义，具有静态存储期，零初始化；`extern int b;` 是非定义声明。带初始化器的 `extern int b = 1;` 是定义。
- 普通外部链接全局变量通常在头文件用 extern 声明，在一个 cpp 中定义。多个 cpp 定义同一变量违反 ODR，通常链接报错。
- 普通非静态成员属于每个对象；静态成员由对象共享。`static int total;` 在类内通常只是声明，类外用 `int Counter::total = 0;` 定义。
- C++17 的 `inline static int total = 0;` 可在头文件中定义，多个翻译单元共享同一实体。
- pragma once 只防止同一翻译单元重复包含，不能阻止多个 cpp 各自出现定义。

## 追问链
1. 没初始化就是声明吗？→ 不是，int a 也可以是定义。
2. 类静态成员每个 cpp 一份吗？→ 不是，static 在这里表达类级共享。
3. 两个 cpp 都定义静态成员？→ 普通非 inline 定义重复，违反 ODR。

## 项目证据 / 反例

- 未提供项目证据；保留对话中的易错点供复习。
