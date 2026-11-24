---
id: cpp/stl/range-algorithms
title: 范围算法、查找与去重
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

范围算法、查找与去重的接口约定、使用条件与易错点。

## 机制

- 范围 [first,last) 包含 first，排除 last。find 未找到返回传入的 last。erase(pos) 要求实际元素，erase(first,last) 允许空范围及 end 作为右边界。
- vector 的 erase 返回删除区间之后的新位置。遍历删除时用 it = v.erase(it)，保留时才 ++it。insert 返回新插入元素的位置，可在 end 插入。
- sort 接收随机访问范围及比较器。comp(a,b) 为真表示 a 应排在 b 前，不能用 >=；相同排序键需保持原顺序时用 stable_sort。
- remove/remove_if 把保留元素稳定集中到前缀，返回逻辑终点。尾部仍存在但值未指定；erase(newEnd,end) 才缩短容器。C++20 可用 std::erase / std::erase_if 简化 vector 删除。
- unique 只合并相邻的等价元素。全局去重且允许改变顺序时，使用 sort → unique → erase。
- 升序范围中 lower_bound 返回第一个 >= 目标的位置，upper_bound 返回第一个 > 目标的位置。判断存在需先检查边界再比较值。lower_bound 也可作为有序插入位置。

## 追问链

1. remove 后能读第四、第五项吗？→ 若仍在 size 内可以，但不能依赖尾部值。
2. lower_bound 找到位置就表示键存在？→ 还要检查等价。
3. 为什么 erase 后不能无条件 ++it？→ 会漏检补位的元素，或越过 end。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

