---
id: cpp/object-model/overload-override-and-hiding
title: 重载、重写、隐藏与 using
level: null
level_status: 待用户确认
reviewed: 2026-11-01
next_review: 2026-11-03
projects: []
sources: [本轮聊天整理]
---

## 一句话

重载按参数选择，重写参与虚分派，隐藏影响名字查找，三者不能混淆。

## 机制
- 重载：同一作用域同名函数，参数列表有允许的区别；仅返回类型不同不构成重载。
- 重写：派生类对应基类虚函数，参数类型、成员 const、引用限定等须匹配；返回类型须相同或满足协变规则，建议 override 检查。
- 隐藏：派生同名声明可隐藏基类同名重载，无须基类 virtual；不会因为参数不匹配就自动回头查找基类。
- using Base::show 引入基类整组同名重载，让其参与查找；编译期重载决议选择最佳候选，若选中虚函数再进行动态分派。
- d.Base::show(1) 显式限定基类实现，抑制这次虚分派。
- using std::cout 引入名字；using namespace std 是命名空间 using 指令；using Name=Type 是类型别名；template<class T> using List=vector<T> 是别名模板，不是新类模板；using Base::Base 继承构造函数。
- 类模板 template<class T> class Box {...} 定义类型家族，Box<int> 与 Box<double> 是不同类型；定义通常须在实例化点可见，常放头文件。
- 用户描述 JSON 元素反序列化由多态工厂选择具体类型。模板可复用流程但并非必需，运行时 type 字符串选择仍需工厂等机制；未看源码不认定其具体模板形式。

## 追问链
1. show(int) 与 show(double) 是重写？→ 不是。
2. using 如何知道选哪个？→ 引入候选后按实参重载决议。
3. 返回类型不同一定不能重载？→ 仅返回不同不行，参数有合法区别则可以。

## 项目证据 / 反例
- 用户口述项目用多态工厂创建 JSON 对应元素、公差、坐标系；未提供代码，不补写实现细节。

## 关联
- [[cpp/object-model/virtual-dispatch]]
