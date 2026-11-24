---
id: cpp/memory/dynamic-memory-and-lifetime
title: 动态内存与悬空指针
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

普通指针不管理资源生命周期；分配与释放必须配对，释放后别名仍可能悬空。

## 机制
- new/delete 创建销毁对象，处理构造析构；malloc/free 分配释放内存，不调用构造析构。
- 普通 new 失败抛 bad_alloc；nothrow 分配失败返回空指针。malloc 失败返回空指针，返回类型为 void*，C++ 转成具体对象指针需显式转换。
- new int 没有初始化整数值，new int(10) 为 10，new int() 为 0。
- 配对：new/delete、new[]/delete[]、malloc/free。数组无需逐项手动 delete；错配释放是未定义行为。
- delete 不会自动清空指针；重复释放和解引用悬空指针是未定义行为，不保证抛出可捕获异常。delete 空指针合法。
- 覆盖唯一持有地址的指针、或普通指针离开作用域而未释放动态对象，会泄漏。
- 局部变量生命周期结束后，返回其地址或引用都会悬空。局部 static 具有静态存储期，但名字作用域仍局限于函数。

## 追问链
1. p、q 同指一块内存，delete p 后 p=nullptr，q 呢？→ 仍悬空。
2. 指针局部变量销毁会 delete 所指对象？→ 不会。
3. new 与 malloc 可混用释放吗？→ 不可以。

## 关联
- [[cpp/memory/raii]]

## 项目证据 / 反例

- 未提供项目证据；保留对话中的易错点供复习。
