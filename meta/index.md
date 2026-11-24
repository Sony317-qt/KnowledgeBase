# 知识库仪表盘

> 只维护"该看什么"，不维护内容本身。内容在 `tech/` 与 `projects/`。
> 2026-10-30：已整理本轮 C++ 对话，level 待用户确认，未自动评定。

## 一、复习队列（本周）

| 技术点 | level | 上次 | 下次 | 状态 |
|---|---|---|---|---|
| [[cpp/basics/declaration-and-definition]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/basics/pointer-reference-and-const]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/basics/c-string-copy]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/memory/dynamic-memory-and-lifetime]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/object-model/initialization-and-destruction]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/memory/raii]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/object-model/rule-of-five]] | 待确认 | 2026-10-30 | 2026-11-01 | 待复习 |
| [[cpp/memory/noexcept-and-move]] | 待确认 | 2026-10-30 | 2026-11-01 | 已补移动操作；值类别待系统讲解 |

## 二、本周待补

2026-11-05 已整理本轮 STL 内容，掌握等级待确认。

| 技术点 | level | 上次 | 下次 | 状态 |
|---|---|---|---|---|
| [[cpp/stl/vector]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/range-algorithms]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/ordered-associative]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/unordered-associative]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/sequence-containers]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/container-adapters-and-heaps]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |
| [[cpp/stl/iterators]] | 待确认 | 2026-11-05 | 2026-11-07 | 已讨论，待复习 |

2026-11-02 补充整理：[[cpp/object-model/initialization-and-destruction]]：析构异常、noexcept、delete/delete[] 配对与虚析构；level 仍待确认，下次复习 2026-11-04。本次仅整理该主题，vector/set 对话尚未整理。

2026-11-01 新整理（level 均待确认，下次复习 2026-11-03）：

- [[cpp/object-model/encapsulation-and-inheritance]]：封装、继承权限、class/struct。
- [[cpp/object-model/memory-layout-and-alignment]]：对齐计算、嵌套结构体、空基类优化。
- [[cpp/object-model/virtual-dispatch]]：多态、虚表模型、生命周期、切片。
- [[cpp/object-model/overload-override-and-hiding]]：重载、重写、隐藏、using 与模板入门。

本轮新增复习：

- [[cpp/memory/unique-ptr]]：独占、get/reset/release、函数传参、数组和删除器；level 待确认，2026-11-01 复习。
- [[cpp/memory/shared-ptr]]：共享计数、控制块、线程安全、shared_from_this；level 待确认，2026-11-01 复习。
- [[cpp/memory/weak-ptr]]：lock、expired、循环引用与销毁时序；level 待确认，2026-11-01 复习。

> 清空重开后的第一批。补完一条就建一个文件，在这里划掉，并更新第五节的统计。

1. `cpp/concurrency/condition-variable` —— 为什么必须配 mutex、虚假唤醒
2. `cpp/concurrency/memory-order` —— volatile 能不能同步、acquire/release
3. `cpp/memory/zero-copy` —— QImage/Mat 零拷贝、`cv::Mat` 包外部内存
4. `video/rtsp-rtp` —— 取流协议与 TCP/UDP 取舍
5. `video/h264-nalu` —— NALU / I 帧 / GOP
6. 第一个项目接入后，按 `workflow.md` 建 `projects/<name>/README.md`

## 三、项目清单

| 项目 | 状态 | 缺口 |
|---|---|---|
| —— | —— | 空 |

## 四、强项（可主动引导面试官）

> 随 level 4/5 的条目增加而更新。

（暂无）

## 五、统计

- 技术点：22（已填充；不计空白占位文件）
- 项目：0
- 平均 level：——

## 六、本轮易错点

- q 与 &q：保存的地址和指针自身地址不同；普通参数复制指针，不能直接修改外部指针。
- const 指针限制访问路径；返回引用后用普通变量接收仍然得到独立值。
- new[] 必须配 delete[]；重复释放属于未定义行为，不保证抛异常。
- 构造顺序按声明；构造失败只清理已完成部分，先清理再进入 catch。
- 拷贝构造和拷贝赋值不同；默认移动裸指针不会自动转移所有权。
- 当前进度：智能指针基础已聊完一轮，重点复习借用与所有权、weak_ptr 失效和循环关系的销毁顺序；const shared_ptr 的复制与修改问题尚未完成问答。值类别已回顾，引用折叠与完美转发待后续。
- 封装、继承、布局和运行时多态已讨论一轮。重点复习 sizeof 不等于 alignof、默认参数看静态类型、智能指针转换不切片。
- STL 已讨论一轮。重点复习容量与元素数量、半开区间、键等价、迭代器失效、堆的有效范围。详见上方 STL 条目。
