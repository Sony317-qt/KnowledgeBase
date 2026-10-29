# 知识库仪表盘

> 只维护"该看什么"，不维护内容本身。内容在 `tech/` 与 `projects/`。

## 一、复习队列（本周）

| 技术点 | level | 上次 | 下次 | 状态 |
|---|---|---|---|---|
| [[cpp/object-model/virtual-dispatch]] | 3 | 2026-10-29 | 2026-11-02 | 待复习 |
| [[cpp/memory/noexcept-and-move]] | 2 | 2026-10-29 | 2026-10-31 | 待复习 |
| [[cpp/object-model/rule-of-five]] | 2 | 2026-10-29 | 2026-10-31 | 待复习 |

## 二、本周待补（来自 2026-10-29 的面试暴露）

按优先级排，补完一条就建一个文件：

1. `cpp/concurrency/condition-variable` —— 为什么必须配 mutex、虚假唤醒
2. `cpp/concurrency/memory-order` —— volatile 能不能同步、acquire/release
3. `cpp/memory/zero-copy` —— QImage/Mat 零拷贝、`cv::Mat` 包外部内存
4. `video/rtsp-rtp` —— 取流协议与 TCP/UDP 取舍
5. `video/h264-nalu` —— NALU / I 帧 / GOP
6. `projects/industrial-vision-measure` 的数字补齐（帧率、精度、节拍）

## 三、项目清单

| 项目 | 状态 | 缺口 |
|---|---|---|
| [[projects/industrial-vision-measure/README]] | 骨架已建 | 量化数字、算法细节待补 |

## 四、强项（可主动引导面试官）

> 随 level 4/5 的条目增加而更新。

- 多线程架构拆分（采集 / 测量 / 渲染）与帧交接策略 —— 有实战证据
- 工业测量链路：相机 SDK → 缓冲 → 算法 → 拟合

## 五、统计

- 技术点：3
- 项目：1
- 平均 level：2.3
