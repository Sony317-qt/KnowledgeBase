---
id: cpp/stl/iterators
title: 迭代器能力与边界
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

迭代器能力与边界的接口约定、使用条件与易错点。

## 机制

- vector、array、deque 支持随机访问，可以 it+n 和相减，适配 std::sort。deque 随机访问但存储不连续。vector<bool> 是 vector 连续存储规则的特例。
- list、map、set 及有序 multi 版本支持双向遍历。forward_list 和 unordered 容器保证向前遍历。适配器没有公开迭代器。
- std::next(it,n) 和 std::prev(it,n) 返回移动后的副本，默认走一步。prev 需要双向迭代器；链表逐步移动，随机访问迭代器可以直接定位。
- end 表示尾后边界，不可解引用。非空双向范围可从 end 回退到最后一个元素；不能从 begin 回退。forward_list 的 before_begin 是专用哨兵。
- 无符号倒序下标从 size 开始，条件 i>0，访问 v[i-1]，避免空容器的 size()-1 和 0 回绕。范围 for 缓存终点；auto 拷贝元素，auto& 引用元素，const auto& 限制通过该引用修改。

## 追问链

1. list 能直接 std::sort 吗？→ 用成员 sort，标准 sort 要求随机访问。
2. prev(begin()) 合法吗？→ 不合法。
3. next(it) 后 it 自己变化吗？→ 不变，返回新位置。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

