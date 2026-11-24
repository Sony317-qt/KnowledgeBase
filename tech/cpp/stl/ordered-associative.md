---
id: cpp/stl/ordered-associative
title: 有序关联容器与等价范围
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

有序关联容器与等价范围的接口约定、使用条件与易错点。

## 机制

- set/map 默认 std::less<Key>，按键有序；通常由平衡树实现。map 的键只读，值可改；set 元素就是键。比较器双向均为 false 时视为等价，即使 == 不同。
- set/map 每个等价键最多一个元素。multiset/multimap 允许多个等价键。成员 find 利用树结构；contains 从 C++20 起可用于判存在。
- equal_range 返回 [lower_bound,upper_bound) 的迭代器对。默认升序时分别为第一个 >= 和第一个 >。未匹配时两者相等，可能都指向下一较大键，也可能都等于 end。
- 插入保留迭代器，删除仅使被删元素的位置失效。erase(key) 删除所有等价项并返回数量；erase(it) 只删一项并返回下一位置。按年龄分组后可在范围内按姓名筛选。
- map 的 [] 缺键时插入值初始化项，at 缺键抛异常，find 缺键返回 end。multimap 没有 [] 和 at。map<Key,vector<T>> 可按键保存整组；清空 vector 不删 map 的键。
- C++17 的 try_emplace 只在缺键时构造值，但调用前仍会求值实参。insert_or_assign 遇已有键更新；返回的 bool 只表示是否新增。普通 insert 对重复键保留旧记录。
- C++17 可 extract 节点后改键再插回；先检查节点非空。若新键冲突，非提示版本 insert 的结果 node 持有未插入节点，可改回原键恢复。

## 追问链

1. 10:30 有两条记录，equal_range 会包含两条吗？→ 都在左右边界之间。
2. try_emplace(name, calculateScore()) 会跳过计算吗？→ 不会；需先查找再决定是否计算。
3. 改键插入失败，丢弃返回值会怎样？→ 取出的节点会被销毁。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

