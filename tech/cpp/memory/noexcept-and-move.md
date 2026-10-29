---
id: cpp/memory/noexcept-and-move
title: noexcept 与移动语义
level: 2
tags: [cpp, move, exception, interview]
aliases: [noexcept, move_if_noexcept, 移动语义]
reviewed: 2026-10-29
next_review: 2026-10-31
projects: [industrial-vision-measure]
sources: [cppreference, cppcoreguidelines C.66]
---

## 一句话

`noexcept` 是"这个函数不抛"的**编译期契约**：容器据此决定敢不敢用你的移动，编译器据此省掉异常清理代码；写挂了会把异常升级成 `std::terminate`。

## 机制

- 两种用法：函数说明符 `void f() noexcept;` / 运算符 `noexcept(expr)` → `bool`
- 违约后果不是 UB，是**直接 terminate**（进程死）
- 标准库策略 `move_if_noexcept`：**移动构造是 noexcept 才用移动**，否则退回拷贝
  （因为扩容要强异常保证：搬一半失败必须能回滚）
- 默认情况：析构函数天然 `noexcept`；`= default` 的移动构造的 noexcept 由成员推导

## 追问链

1. **不写 noexcept 会怎样？**
   `std::vector<Frame>` 扩容时退化为拷贝 —— 6 MB 一帧 × 1000 帧 = 6 GB 搬运，而不是几 KB 的指针搬运。
2. **写了 noexcept 但成员移动会抛呢？**
   标准规定这个显式 default 的移动构造**被定义为 deleted**（不是编译错误），
   你在别处一用就得到 "use of deleted function"，极难排查。
3. **移动之后源对象什么状态？**
   有效但未指定（valid but unspecified）：能析构、能重新赋值，但不能假设里面还是原值。
4. **怎么在编译期验证？**
   `static_assert(noexcept(Frame(std::declval<Frame&&>())));`

## 项目证据 / 反例

- 工业测量项目：帧对象在 `std::vector` 里攒批处理，移动标了 `noexcept` 才避免每次扩容深拷贝 6 MB。
- **反例**："标记了 noexcept 就会更快" ✗ —— 它不改变函数体执行速度，
  改变的是**调用方敢不敢用移动**、以及**异常表要不要生成**。
- **反例**：以为 `= default` 的移动一定是 noexcept ✗ —— 由成员推导，成员会抛就会抛（强写 noexcept 反而 deleted）。

## 关联

- [[cpp/object-model/rule-of-five]]
