---
id: cpp/memory/raii
title: RAII 与智能指针选择
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

将资源所有权交给对象，利用对象析构自动清理资源。

## 机制
- 已构造完成的局部 RAII 对象在正常离开作用域、return、异常栈展开时析构；构造未完成的整体不会执行自身析构。
- 若裸指针成员 new 成功后构造函数体抛异常，默认清理指针成员不会 delete 所指内存。unique_ptr 成员已构造完成则能在失败清理时释放。
- RAII 不限智能指针：string、vector、锁守卫都管理资源。优先直接存对象，动态独占用 unique_ptr，确有共享所有权才用 shared_ptr，弱观察可用 weak_ptr。
- 多处使用不等于共享所有权；借用通常用引用或非拥有裸指针。shared_ptr 有计数开销，循环强引用可能泄漏。
- QImage、QPixmap 是 RAII 值类型，使用隐式共享；普通复制通常共享底层数据，正常修改接口按需分离，最后持有者释放。无需仅为自动释放再套智能指针。
- QImage 包装外部缓冲区默认不接管该缓冲区，须保证缓冲区寿命或安排清理回调；不能把包装对象的 RAII 等同于所有外部资源都被接管。

## 追问链
1. 第一个智能指针成员成功、第二个抛异常？→ 第一个析构释放资源。
2. 智能指针越多越好？→ 不是，先明确所有权及是否需动态对象。
3. 允许复制的 RAII 类型一定重复释放？→ 不会，取决于复制与共享机制。

## 关联
- [[cpp/memory/dynamic-memory-and-lifetime]]
- [[cpp/object-model/rule-of-five]]
- [[cpp/memory/unique-ptr]]
- [[cpp/memory/shared-ptr]]
- [[cpp/memory/weak-ptr]]

## 项目证据 / 反例

- 未提供项目证据；保留对话中的易错点供复习。
