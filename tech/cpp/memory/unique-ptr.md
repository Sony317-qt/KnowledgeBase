---
id: cpp/memory/unique-ptr
title: unique_ptr：独占、借用与转交
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

unique_ptr 是独占资源的类模板，利用析构释放资源，禁止复制但支持移动。

## 机制
- auto p=std::make_unique<int>(10) 创建管理对象；*p、p-> 用于访问。get() 返回借用裸指针，可以读写非 const 对象，但不能擅自 delete，所有权仍属于 p。
- reset() 释放旧对象并置空；reset(new int(20)) 更换对象，旧对象被释放。reset(p.get()) 会留下悬空指针，禁止这样写。
- release() 置空自身、返回裸指针，不释放对象。调用者接管清理责任，丢失返回地址可能泄漏。转交给另一个 unique_ptr 通常直接 std::move。
- std::move 后源 unique_ptr 为空；源对象仍存在。不能用 p.get() 再创建另一个独占拥有者，否则重复释放。
- 参数 int& / int* 是借用所指对象；unique_ptr<T> 按值参数接收所有权，调用时 std::move(p)，p 不会在返回时自动恢复。unique_ptr<T>& 引用外部智能指针，允许重置或更换资源。
- get() 的借用指针不会在 reset 或析构时自动清空；原对象销毁后它悬空。
- const unique_ptr<int> 限制指针对象本身，仍可修改整数；unique_ptr<const int> 限制所指整数，指针仍可 reset。
- 数组用 unique_ptr<int[]>，默认 delete[]；make_unique<int[]>(5) 的整数元素初始化为零。长度管理和扩容通常优先 vector。
- 自定义删除器可管理文件：unique_ptr<FILE, FileCloser>，FileCloser 内调用 fclose。资源非空时析构调用对应删除器；不能用默认 delete 释放文件句柄。

## 追问链
1. get 返回的裸指针能改值吗？→ 可以，若所指类型非 const；不能接管释放。
2. take(unique_ptr<int> value) 结束为何 p 不恢复？→ value 是移动构造的新拥有者，析构释放，源 p 仍为空。
3. release 与 reset？→ 前者转交清理责任、不释放；后者清理旧对象。

## 项目证据 / 反例
- 无项目证据。对话曾混淆“借用指针不能修改值”和“不能释放资源”；曾误以为按值移动传参后外部仍持有资源，已逐步澄清。

## 关联
- [[cpp/memory/raii]]
- [[cpp/memory/noexcept-and-move]]
