# C++ 面试冲刺计划（2026-10-29 起，一周）

> 目标岗位：视频监控软件 · C++ · 应届
> 每天 20–30 分钟，产出物必须是**知识库里的一两个 md 文件**（不是"看完了"）。

| 天 | 主题 | 产出物（文件） | level 目标 |
|---|---|---|---|
| D1 | 对象模型：虚函数 / 虚析构 / 对象布局 | `virtual-dispatch` ✅、`rule-of-five` ✅ | 3 |
| D2 | 内存与 RAII：智能指针、所有权、悬垂 | `shared-ptr-thread-safety`、`raii` | 3 |
| D3 | 拷贝与移动：五法则、`noexcept` | `noexcept-and-move` ✅、补 `zero-copy` | 3 |
| D4 | 并发：mutex / 条件变量 / atomic / 内存序 | `condition-variable`、`memory-order` | 2→3 |
| D5 | STL：容器复杂度、迭代器失效、扩容 | `vector-growth`、`iterator-invalidation` | 3 |
| D6 | 视频链路：TCP/UDP、RTSP/RTP、H.264 NALU | `rtsp-rtp`、`h264-nalu` | 2 |
| D7 | 项目复盘 + 二次模拟面 | 补全 `projects/industrial-vision-measure` | 4 |

## 每天的三步

1. **5 分钟**：过 `meta/index.md` 的复习队列（合上笔记复述）
2. **15 分钟**：一个主题，写或改一个文件（重点是 `## 追问链`）
3. **5 分钟**：口述一遍"一句话 + 三层追问"，录音回放

## 面试前一晚清单

- [ ] 项目的三个数字能脱口而出（帧率、精度、节拍）
- [ ] 三次重构各解决什么问题，一句话一个
- [ ] 一个"我没答好、后来补上了"的故事（真实，比完美答案加分）
- [ ] 反问面试官的问题 2 个（团队的技术难点 / 代码 review 文化）
