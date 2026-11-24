---
id: cpp/stl/unordered-associative
title: 哈希容器、分桶与组合键
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

哈希容器、分桶与组合键的接口约定、使用条件与易错点。

## 机制

- unordered_set 只存键，unordered_map 存键值对；multi 版本允许等价键重复，遍历时等价项相邻，equal_range 返回整组。无键排序和 lower_bound；unordered_multimap 无 [] 和 at。
- 哈希值用来定位桶，桶中按键的相等规则查找。取模是理解模型，具体桶映射由实现决定。查找平均 O(1)，最坏 O(n)。
- 负载因子是元素总数除以桶数，重新分桶看全表而非单桶上限。rehash 改变桶布局，使旧迭代器失效，已有元素的引用和指针仍有效。
- reserve(n) 按预计元素总数准备桶，可能触发 rehash。先 reserve，再保存迭代器。
- 组合键可用 senderId 和 messageId 定义相等。相等的键必须有相同哈希，哈希相同的键仍可不相等。只哈希 messageId 合法但可能让同编号大量集中，增加桶数无法分开哈希值相同的键。
- 唯一键容器 insert 返回 pair<iterator,bool>，C++17 可用 auto [it, inserted] 拆开。重复时迭代器指向旧元素，bool 为 false。map 型统计用 ++counts[id]，只查询则用 find 避免新增零计数。

## 追问链

1. {1,42} 和 {2,42} 哈希相同能共存吗？→ 能，键不等。
2. 第三次插入 {1,42} 的 second 是什么？→ false，表示未新增。
3. rehash 后怎样继续用旧迭代器对应的记录？→ 按键重新 find。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

