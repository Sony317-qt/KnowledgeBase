---
id: cpp/object-model/virtual-dispatch
title: 虚函数分派与虚析构
level: 3
tags: [cpp, oop, interview]
aliases: [动态绑定, vtable, 虚表, vptr]
reviewed: 2026-10-29
next_review: 2026-11-02
projects: [industrial-vision-measure]
sources: [cppreference, Effective C++ Item 7]
---

## 一句话

虚函数用"对象里的 vptr + 每个类一张 vtable"换运行期多态，代价是一次间接跳转、对象多一个指针、调用通常不能内联。

## 机制

- vtable 编译期生成，**每个类一份**（不是每对象一份），放只读数据段
- 对象里多一个隐藏的 vptr 指向它（实现上常在对象起始处，标准不规定位置）
- `p->f()` 展开为：取 vptr → 取槽位函数指针 → 间接调用
- 构造/析构期间 vptr 被设成**当前类自己的表** → 所以构造里调虚函数没有多态

## 追问链

1. **为什么基类析构要 virtual？**
   `Base* p = new Derived; delete p;` 若 `~Base` 非虚 → 只调 `~Base`，派生成员不析构 → UB/泄漏。
2. **构造函数里调虚函数有多态吗？**
   没有。此时派生类部分尚未构造，vptr 指向基类表。
3. **对象多大？空类加一个虚函数呢？**
   空类 1 字节（保证地址唯一）；加虚函数变 8 字节（64 位，只有一个 vptr）。
4. **虚函数能内联吗？**
   一般不能；编译器在能确定动态类型时可以做去虚化（devirtualization），`final` 也能帮上忙。

## 项目证据 / 反例

- 工业测量项目：`CImgCircleFrame` 作为多态基类持有图像缓冲，析构必须 virtual ——
  否则 `delete` 基类指针时派生类缓冲不释放，内存缓慢上涨。
- **反例**："虚表存在对象里" ✗ —— 表是每类一份，对象里只有指针。
- **反例**：只在**子类**析构上加 virtual ✗ —— 通过基类指针 `delete` 时看的仍是基类的析构，
  救不了。真正必须有 virtual 的是基类；子类上应该写 `override` 来让编译器校验。

## 关联

- [[cpp/object-model/rule-of-five]]
