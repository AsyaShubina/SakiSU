# SakiSU

<img align="right" src="docs/SakiSU_blue.svg" width="220px" alt="SakiSU Icon">

**简体中文** | [English](docs/README.md) | [vivo/iQOO 适配教程](docs/zh/vivo.md)

SakiSU 是基于 [ReSukiSU](https://github.com/ReSukiSU/ReSukiSU) 的下游分支，保留 KernelSU/SukiSU 系 root 管理、模块系统和 App Profile，并重点补充 vivo/iQOO 设备上的内核级 root 适配。

当前同步策略是：以 ReSukiSU 最新 `main` 为底，按主题重放 SakiSU 自己的改动。测试先进入 `dev` 或同步分支，稳定后再合入 `main`。

## 重点功能

- KernelSU/SukiSU 系内核级 `su` 与 root 授权管理。
- 模块系统、App Profile、SuSFS/tracepoint 等上游能力。
- vivo/iQOO 兼容模式：同一个开关用于“去除vr或适配vivo特性”。
- vivo `vendor_boot` 路径只移除 `vr.ko`，不会注入 KernelSU LKM。
- vivo `init_boot`/boot 路径继续注入 LKM，并优先使用 `_vivo` KMI/LKM 变体。
- CI 保留长期签名密钥优先、临时同批次签名兜底的构建流程。

## vivo/iQOO 快速说明

Manager 中的 vivo 开关含义是 **“去除vr或适配vivo特性”**。

- 选择 `vendor_boot.img` 时，SakiSU 会自动识别 vendor ramdisk，清理 `vr.ko` 及其 `modules.*` 引用，不弹出不必要的 KMI 选择，也不注入 LKM。
- 选择 `init_boot.img` 或兼容 boot ramdisk 时，SakiSU 走正常 LKM 注入流程，并优先选择 `_vivo` KMI/LKM。
- rmvr 和 LKM 注入互不依赖。可以只修补 `vendor_boot` 去除 `vr.ko`，也可以只修补 `init_boot` 注入 KernelSU。

完整背景、风险说明和教程见 [docs/zh/vivo.md](docs/zh/vivo.md)。

## 文档

- [中文文档](docs/zh/README.md)
- [English documentation](docs/README.md)
- [vivo/iQOO 适配教程](docs/zh/vivo.md)
- [vivo/iQOO compatibility guide](docs/vivo.md)
- [vivo 实现记录](DEVLOG-VIVO.md)
- [上游同步注意事项](SAKISU-UPSTREAM-SYNC.md)

## 鸣谢

- [ReSukiSU/ReSukiSU](https://github.com/ReSukiSU/ReSukiSU)：当前上游基底。
- [SukiSU-Ultra/SukiSU-Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)：上游血统。
- [KernelSU](https://github.com/tiann/KernelSU)：内核级 root 方案基础。
- 感谢参与 vivo/iQOO 反 root 研究与验证的社区贡献者。

## 许可证

`kernel` 目录遵循 GPL-2.0-only；其余部分按仓库内许可证声明执行。
