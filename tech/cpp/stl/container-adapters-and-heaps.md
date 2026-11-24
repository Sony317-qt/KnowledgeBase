---
id: cpp/stl/container-adapters-and-heaps
title: 容器适配器与堆算法
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

容器适配器与堆算法的接口约定、使用条件与易错点。

## 机制

- stack 后进先出，默认 deque；top 可修改栈顶，push/emplace 添加，pop 删除且返回 void。queue 先进先出，默认 deque，也可用 list；front/back 访问两端，push/emplace 加队尾，pop 删队首。
- 三种适配器有 size、empty、swap，无公开下标或迭代器。C++23 有 push_range。读取端点或 pop 前要求非空；需要保留值时先复制或在允许时移动，删除后原元素的引用悬空。
- priority_queue 默认 vector + less，大根堆；greater 是小根堆。top 返回只读引用，pop 删除堆顶。比较器为真表示左操作数优先级较低。相同优先级没有 FIFO 保证，可用递增序号打破平局。
- lambda 比较器可用 decltype(cmp) 指定类型，并传 cmp 给构造函数。已有 vector 可用构造函数移入并线性建堆。
- make_heap 建堆；push_back 后用 push_heap 把新尾元素纳入堆；pop_heap 将旧堆顶放到末尾，前缀仍是堆，随后 pop_back 才删除元素。
- 连续 pop_heap 时要删掉已弹出的尾元素或逐次缩短操作范围。sort_heap 由已有堆生成有序范围，默认大根堆结果升序。is_heap 检查堆性质；各操作应使用同一比较规则。
- 小根堆保留最大的 k 个数时，可在超过 k 项后弹掉最小候选。

## 追问链

1. pop_heap 后 back 为何是旧堆顶？→ 它被搬到容器末尾，size 不变。
2. 小根堆的任务比较器怎样写？→ a.deadline > b.deadline，较早者先出队。
3. queue 的 pop 后还能读之前保存的 front 引用吗？→ 被删除元素的引用已悬空。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

