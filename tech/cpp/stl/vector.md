---
id: cpp/stl/vector
title: vector 的容量、元素与失效
level: null
level_status: 待用户确认
reviewed: 2026-11-05
next_review: 2026-11-07
projects: []
sources: [本轮 STL 学习对话]
---

## 一句话

vector 的容量、元素与失效的接口约定、使用条件与易错点。

## 机制

- size 是活跃元素数，capacity 是无需重新分配可容纳的数量。reserve 只增容量，resize 增删尾部元素，新增 int 默认是 0。resize 缩小、clear、pop_back 保留容量。shrink_to_fit 是非强制缩容请求。
- 下标、at、data 可访问的元素范围都以 size 为准。at 越界抛 out_of_range，front/back/pop_back 要求非空。data 借用连续存储，不能 delete[]；const vector 的 data 返回 const 指针。
- 重新分配使旧指针、引用、迭代器全部失效。未扩容的尾插保留原元素位置，但旧 end 失效；中间插入使插入点及之后失效，erase 使删除点及之后的迭代器和引用失效。assign 替换全部并使旧位置失效。swap 保留元素位置，元素改属另一容器。
- push_back 均摊 O(1)，扩容单次 O(n)，增长倍率无固定保证。已知总数时可一次 reserve，避免每次 reserve(size()+1)。reserve 应在保存迭代器前调用。
- push_back(x) 对同类型左值通常拷贝构造，std::move(x) 通常允许移动构造。emplace_back 转发构造参数，传已构造对象仍可能拷贝或移动；C++17 起返回新元素引用，push_back 返回 void。二者都可能扩容。
- 圆括号 (3,7) 产生三个 7，花括号 {3,7} 列出两个元素。C++17 支持 vector{1,2,3} 推导 vector<int>。范围构造使用左闭右开区间。== 比较元素与顺序，关系比较按字典序。
- 拷贝 vector 会拷贝元素，引用传参借用原容器，const 引用限制通过它修改。逐个移动 unique_ptr 保留源 vector 的大小但源元素变空；移动整个 vector 后源对象仍有效，通常按状态未指定使用。std::move 只转换表达式，不去掉 const。

## 追问链

1. 保存 begin 后追加十万次还能解引用吗？→ 发生扩容后不行，重新取 begin 或事先 reserve。
2. reserve 后为何不能写预留位置？→ 未增加 size。
3. 范围 for 内尾插，预留容量就安全？→ 旧 end 仍失效。要追加原元素可先保存原 size，用下标遍历固定次数。

## 项目证据 / 反例

- 当前例子来自学习对话，尚无项目证据。

