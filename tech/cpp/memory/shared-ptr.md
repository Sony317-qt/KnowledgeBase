---
id: cpp/memory/shared-ptr
title: shared_ptr：控制块与共享自身
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

复制 shared_ptr 共享控制块而不是复制所指对象，最后一个强拥有者负责销毁对象。

## 机制
- make_shared<T>() 创建共享对象；auto b=a 或 b=a 共享既有控制块。复制增加强计数；移动把拥有者位置交出去，不增加总强计数。
- 空 shared_ptr 的 use_count() 为 0；两份共享拥有者各读到 2。reset 放弃自己的所有权，不会自动清空其他指针。
- 控制块记录强引用计数、弱引用相关信息及销毁所需信息；具体地址布局、删除器和分配器存储由实现决定。内部弱计数不一定等于 weak_ptr 数量。
- 强计数归零时对象生命周期结束；仍有弱观察者则保留控制块。make_shared 可能合并分配对象存储和控制块，对象销毁不代表整块分配立即归还。
- 同一裸指针分别构造两个 shared_ptr 会产生独立控制块，各自计数 1，导致重复销毁等未定义行为。
- 对象内 std::shared_ptr<Task>(this) 同样会另建控制块。应公开继承 enable_shared_from_this<Task>，用 shared_from_this() 返回共享已有控制块的指针。
- shared_from_this 调用时必须已建立共享所有权；栈对象或构造函数内调用不能依赖它，未关联控制块时抛 bad_weak_ptr。
- 不同 shared_ptr 实例共享控制块时可在不同线程各自 reset，最后一次释放只销毁一次；同一个 shared_ptr 实例并发读写需同步或合适的 atomic<shared_ptr<T>>。
- 控制块安全不等于对象数据安全：同时 ++ 同一个普通 int、同时 push_back 同一 vector 仍需同步。use_count 不能充当并发同步依据。

## 追问链
1. shared_ptr(raw) 两次为何不是共享？→ 两次建立控制块，没有复制既有所有权。
2. self 返回后 q 会自动为空吗？→ 不会，返回值在外部继续拥有资源；问题是控制块是否共享。
3. 两线程各自 reset 不同实例？→ 计数维护安全；并发操作同一实例要另外同步。

## 项目证据 / 反例
- 无项目证据。用户已正确回答共享计数、复制和移动的计数变化及 vector 数据竞争；曾误判 self 返回会失效，已澄清。

## 关联
- [[cpp/memory/weak-ptr]]
- [[cpp/memory/raii]]
