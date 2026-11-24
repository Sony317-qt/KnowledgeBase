---
id: cpp/stl/sequence-containers
title: array、deque 与链表
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

array、deque 与链表的接口约定、使用条件与易错点。

## 机制

- array<T,N> 的 N 是类型的一部分。元素连续存储，支持下标、迭代器、data，长度固定。
- deque 通常分块存储，支持常数时间随机访问及两端单项增删；不能把 &d[0] 当作连续数组。两端插入使旧迭代器和 end 失效，已有元素引用与指针保留。
- deque 两端删除只影响被删元素；若删了原末尾，旧 end 也失效。中间插入或删除使全部迭代器、引用和指针失效。
- list 双向，无下标。已知位置时单项增删 O(1)，插入不使旧位置失效，删除只影响被删节点。
- list::splice 把节点接到目标迭代器之前，可转移单节点或整表。迭代器仍指向同一节点，转移后归目标链表。跨链表要求分配器相等。
- list::sort 稳定排序并保留迭代器与引用。merge 按相同规则合并两条已排序的链表并转移节点，跨链表同样要求分配器相等。降序可统一传 [](int a,int b){return a>b;}。
- forward_list 单向，通过 insert_after / erase_after 操作后继。before_begin 可以用作首节点的前驱但不能解引用。无 size、back、push_back；empty 判空，distance 计数需遍历。

## 追问链

1. deque 头插后旧下标和旧引用都跟着元素吗？→ 引用仍绑定原元素，下标表示新布局中的位置。
2. a 的节点 splice 到 b 后谁能 erase(it)？→ b。
3. forward_list 怎样删除首元素？→ pop_front 或 erase_after(before_begin())。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

