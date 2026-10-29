---
id: cpp/object-model/rule-of-five
title: 三/五法则与 = default / = delete
level: 2
tags: [cpp, oop, resource, interview]
aliases: [rule of five, 五法则, 拷贝删除, default delete]
reviewed: 2026-10-29
next_review: 2026-10-31
projects: [industrial-vision-measure]
sources: [Effective C++ Item 5/6, cppcoreguidelines C.21]
---

## 一句话

只要自定义了**析构 / 拷贝构造 / 拷贝赋值**中的任意一个，几乎一定需要另外两个（三法则）；
C++11 再加**移动构造 / 移动赋值**（五法则）。漏写一个，要么 double free，要么对象进不了容器。

## 机制

- `= default`：让编译器生成默认版本 → 决定**平凡性（trivial）**、能否 `constexpr`、异常规范
- 手写 `{}`：用户提供 → 不再是 trivial；内置类型成员**依然不初始化**
- `= delete`：参与重载决议，被选中即编译报错，比"只声明不定义"（链接期才炸）清楚得多
- `= default` 可能被**隐式删除**：成员是引用 / `const` 无初始化器 / 成员不可拷贝

## 追问链

1. **`= default` 等于清零吗？**
   不等于。没有 NSDMI 的内置类型成员仍是垃圾值。要确定归零得靠 `{}` 值初始化或类内 `int x{};`。
2. **成员变量什么时候初始化？**
   基类构造 → 成员按**声明顺序**初始化（不是初始化列表书写顺序）→ 构造函数体。体内是**赋值**不是初始化。
3. **哪些成员必须用初始化列表？**
   `const` 成员、引用成员、没有默认构造的类类型成员。
4. **什么时候必须自己写析构？**
   持有裸资源（`fclose`/`Release()`）时；用 RAII 成员（`unique_ptr`/`vector`）就不必写。

## 项目证据 / 反例

- 工业测量项目：`CImgCircleFrame` 删拷贝、default 移动、虚析构 —— 独占语义 + 可入容器 + 正确多态销毁。
- **反例**：写了 `= default` 就以为有定义可到处用 ✗ —— 若被隐式删除，报的是 "use of deleted function"。
- **反例**：删了拷贝却没写移动 ✗ —— 对象会连 `push_back` 都塞不进去。

## 关联

- [[cpp/object-model/virtual-dispatch]]
- [[cpp/memory/noexcept-and-move]]
