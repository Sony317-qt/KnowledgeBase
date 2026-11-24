---
id: cpp/memory/weak-ptr
title: weak_ptr：观察、lock 与循环引用
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

weak_ptr 不增加强引用计数，访问前用 lock 尝试获得共享所有权，避免所有权环。

## 机制
- weak_ptr 不能直接 * 或 ->；lock 原子地尝试取得 shared_ptr，成功后该局部拥有者保活，失败返回空，不能复活已销毁对象。
- expired 判断强计数是否为零；先检查未过期再操作不能保证期间对象不消失，实际访问优先 if(auto p=w.lock())。
- owner.reset 在 lock 之前且强计数归零，lock 返回空；若先 lock 成功，再 reset owner，局部 p 会让对象继续存活直到 p 释放。
- A、B 双向用 shared_ptr 持有对方，外部释放后各有一个强拥有者，析构根本无法开始，形成泄漏。
- 双向关联不等于双向拥有：next 用 shared_ptr、prev 用 weak_ptr，只观察回向关系。

## 追问链
1. weak_ptr 能让整数继续存在吗？→ 不能，只让观察状态所需控制块保留。
2. A.next=B、B.prev=A，外部 a.reset 后谁销毁？→ A 强计数 1→0 销毁；A.next 释放让 B 从 2→1，外部 b 仍保活 B。
3. 接着 b.reset？→ B 销毁，B.prev 随之销毁；A 无其他弱观察者时控制块可释放，B 控制块也可释放。

## 项目证据 / 反例
- 无项目证据。曾误判 owner.reset 后 lock 仍能保活，也曾误判外部 a.reset 会先销毁 B；保留为复习重点。

## 关联
- [[cpp/memory/shared-ptr]]
