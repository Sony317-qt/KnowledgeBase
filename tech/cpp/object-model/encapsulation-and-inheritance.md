---
id: cpp/object-model/encapsulation-and-inheritance
title: 封装、继承与访问控制
level: null
level_status: 待用户确认
reviewed: 2026-11-01
next_review: 2026-11-03
projects: []
sources: [本轮聊天整理]
---

## 一句话

封装用接口维护有效状态并隐藏实现；继承复用基类部分，访问权限不决定成员是否存在。

## 机制
- private 数据配合校验接口保护不变量；返回可修改引用可能绕过检查。const 成员函数限制经 this 修改普通成员，mutable 可用于内部维护；不代表深层所有状态不可改变。
- public 继承保持基类 public/protected；protected 继承将二者变 protected；private 继承将二者变 private。基类 private 在任何方式下都不可被派生类直接访问，只能经有权调用的接口间接操作。
- 继承方式改变的是经派生类访问的权限，不改基类本身的声明。private 继承时派生类内部仍可调用基类公开接口，普通外部代码不能经派生对象调用。
- class 与 struct 都是类类型，可以互相继承，支持构造析构、虚函数等；默认成员权限及默认继承方式分别为 private 和 public。继承默认值看派生类关键字。
- C 没有内置类、继承和成员访问权限；可用不透明结构体隐藏定义，用独立函数提供接口。

## 追问链
1. private 加无校验 setter 能保护正宽度？→ 不能。
2. 基类私有成员在派生对象中不存在？→ 存在，只是不能直接访问。
3. struct 继承 class 默认什么方式？→ public，与基类关键字无关。

## 项目证据 / 反例
- 卡尺宽度是教学示例，不作为真实项目证据。用户已正确复述访问表，曾混淆成员权限和继承权限。

## 关联
- [[cpp/object-model/virtual-dispatch]]
