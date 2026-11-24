---
id: projects/salticidae/incidents/2026-11-20-ipc-missing-blocks-main-chain
date: 2026-11-20
type: 环境缺陷（P0，阻塞主链路）
---

# 2026-11-20 · IB 部署包缺失，所有抓拍被拒（E-01）

## 症状

启动验证清单里这几行：

```text
[GalaxySDK] 使用 ...\salticidae\GalaxySDK          ✅
[LPC] LInit(...\salticidae) -> 0x0                 ✅
Licensed successfully via SENSE dongle             ✅
[ib] initialize failed: IB 目录不存在:
     E:\GMKTEST\IB_3.8.86_x64_softEnc              ❌ 业务组件缺失
GMKdemo exiting rc=0                                （主动退出，非崩溃）
```

所有抓拍请求以 `kErrCapability` 被拒绝。

## 定位

- IB 目录 `E:\GMKTEST\IB_3.8.86_x64_softEnc` **不存在**。
- IB 包（含 4 个 `.task`：拆图、融合等）属于**手工部署**项，构建不会带出来。
- 顺带：`[ib] initialize failed` 不会拦截启动，只记日志 → 表现为"程序看起来正常，一抓拍就被拒"。

## 根因

两层：

1. **部署项没有纳入构建**：IB 目录、`.task` 文件都是手工拷贝，换机即重来。
2. **失败不暴露**：初始化失败只写日志，启动流程继续，问题被推迟到第一次抓拍才显现。

## 影响

- P0：**主链路完全阻塞**，其他所有验证都无从进行。
- 与另外两条同源风险合并成结论："**当前最大风险是环境不可自举**"——
  IB 包缺失、测试目录缺失、props 依赖本机路径，换机即重来。

## 修法建议

- 索取 IB 3.8.86 完整包（含 4 个 `.task`），纳入部署清单（见 README 部署目录清单）。
- `GMKdemo.ini` 里 `[Fusion] ib_path` / `task_path` 与 `_user.props` 的 `GMKIBDir` 保持一致。
- 启动时把"IB 不可用"提升为界面可见的显式状态，而不是只打一行日志。
- 补 `bootstrap.ps1` 校验 SDK 实体（E-06）。

## 关联

- [[projects/salticidae/README]] —— 启动验证清单、部署目录清单、构建九类报错
- 相邻问题：`.gitignore` 的 `*.json` 误伤参数白名单（T-01）；`test/` 目录不存在（Q-01）
