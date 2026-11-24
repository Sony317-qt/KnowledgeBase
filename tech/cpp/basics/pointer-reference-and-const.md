---
id: cpp/basics/pointer-reference-and-const
title: 指针、引用、const 与参数传递
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

指针保存地址，引用是别名；通过地址修改对象和修改指针自身，是不同操作。

## 机制
- `int a=10; int* p=&a; int** q=&p;`：p 保存 a 的地址，&p 是 p 自己的地址；*p 访问 a；q 保存 p 的地址，*q 访问 p，**q 访问 a。
- `*q=nullptr;` 修改 p；`**q=20;` 修改 a。先把 p 置空后不能再解引用它。
- 普通引用变量必须初始化，不能重新绑定；`int& r=a; r=b;` 是把 b 的值赋给 a。
- `const int* p` 不能通过 p 修改值，但可改指向；`int* const p` 相反；两处 const 则两者均限制。只读访问路径不会让原来的非 const 对象变成常量。
- C 都是值传递。change(q) 复制 q 保存的地址，局部参数 p 改为空不会改变 q；change(&q) 配合 int** 可以修改 q。C++ 的 int*& 参数直接引用外部指针。
- 已有长字符串只读传参通常用 const std::string&，避免复制；小整数通常按值传递。值传递临时对象可能移动或省略复制。
- sizeof(char) 固定为 1；其他大小依实现。常见 64 位 Windows 的 long 为 4 字节，Linux 为 8；long long 通常 8。sizeof(*p) 看被指类型，不是指针大小。

## 追问链
1. p=&b 会改变 a 吗？→ 不会，只换访问对象。
2. swap 中只交换局部指针能交换整数吗？→ 不能，须交换 *x、*y。
3. 返回引用、用 int x 接收？→ x 是独立值，只有 int& 才绑定引用。

## 项目证据 / 反例

- 未提供项目证据；保留对话中的易错点供复习。
