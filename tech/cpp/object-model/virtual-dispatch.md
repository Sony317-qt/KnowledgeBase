---
id: cpp/object-model/virtual-dispatch
title: 运行时多态与虚函数分派
level: null
level_status: 待用户确认
reviewed: 2026-11-01
next_review: 2026-11-03
projects: []
sources: [本轮聊天整理]
---

## 一句话

静态类型决定可见接口和重载选择，动态类型决定虚函数的最终重写实现。

## 机制
- public 继承下，基类指针/引用可访问派生对象的基类部分，不复制对象。派生新增接口不自动对基类指针可见。
- 普通虚函数不使类成为抽象类；存在纯虚最终重写的类是抽象类。派生类可不重写纯虚函数，但仍不能实例化；纯虚函数可有类外定义，纯虚析构必须有定义。
- 重写后仍是虚函数，无须重复 virtual；override 检查匹配；final 函数禁止进一步重写但不取消分派，final 类禁止继承。
- 主流实现使用虚表/vptr，标准不强制。编译链接阶段准备表，构造阶段设置指针；不是每个对象复制一张表。同类型完整对象正常阶段通常共享对应表组，多多态基类可能有多张表及地址调整。
- 构造/析构中的虚调用只到当前构造/析构类，不分派到更派生部分；不能解释成全部构造完成后才设置 vptr。构造函数、static 成员函数不能是虚函数。
- 普通基类指针 delete 派生对象需虚析构，否则 UB，不能只理解成少析构或泄漏。其他虚函数不会自动使析构变虚；基类虚析构会使派生析构也虚。
- 默认参数按静态声明选：Base 的默认值 1、Derived 为 2，Base* 指向 Derived 调用得到 Derived 实现和参数 1。
- Base b=d 创建独立 Base，按基类复制规则切片，调用 Base 实现；原 d 不变。Base&/Base* 不切片。vector<Base> 切片，vector<unique_ptr<Base>> 或 shared_ptr 版本管理完整派生对象，不切片。

## 追问链
1. 派生只写 show(double)，Base* 调 show(1)？→ 没重写 show(int)，调用基类实现。
2. 构造 Base 时调用虚函数？→ Base 阶段实现；析构同理。
3. 智能基类指针是否只保留 Base 数据？→ 不是，完整派生对象仍存在，只限制直接可见接口。

## 项目证据 / 反例
- 无真实项目证据。用户最初误认为有虚函数都不能实例化、虚表在构造时生成；已纠正。多重继承布局仅为常见实现模型。

## 关联
- [[cpp/object-model/overload-override-and-hiding]]
- [[cpp/object-model/memory-layout-and-alignment]]
- [[cpp/memory/shared-ptr]]
