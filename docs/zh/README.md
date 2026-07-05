# SakiSU

<img align="right" src="SakiSU_blue.svg" width="220px" alt="SakiSU Icon">

**简体中文** | [English](../README.md) | [vivo/iQOO 适配教程](vivo.md)

SakiSU 是基于 [ReSukiSU](https://github.com/ReSukiSU/ReSukiSU) 的下游分支，保留 KernelSU/SukiSU 系 root 管理、模块系统、App Profile 等上游能力，并补充 vivo/iQOO 设备上的内核级 root 适配。

当前维护方式是：以 ReSukiSU 最新 `main` 为底，按主题重放 SakiSU 自己的改动。测试先进入 `dev` 或同步分支，稳定后再合入 `main`。

## 重点功能

- 内核级 `su` 与 root 授权管理。
- 模块系统、App Profile、SuSFS/tracepoint 等上游能力。
- vivo/iQOO 兼容模式，开关文案为 **“去除vr或适配vivo特性”**。
- `vendor_boot` 路径只清理 `vr.ko`，不会注入 KernelSU LKM。
- `init_boot` 或兼容 boot ramdisk 路径继续注入 LKM，并优先使用 `_vivo` KMI/LKM。
- CI 构建保留长期签名密钥优先、临时同批次签名兜底的策略。

## vivo/iQOO 快速理解

开启 vivo 修补后，Manager 统一调用：

```text
ksud boot-patch-vivo
```

后端会按镜像内容自动判断路径：

| 镜像 | 行为 |
|---|---|
| `vendor_boot.img` | 移除 `vr.ko` 并清理 `modules.load`、`modules.dep`、`modules.softdep`、`modules.load.recovery`，不注入 LKM。 |
| `init_boot.img` 或 boot ramdisk | 正常注入 KernelSU LKM，并优先选择匹配的 `_vivo` KMI。 |

这两个操作互不依赖。只刷 `vendor_boot` 只代表去除 `vr.ko`；只刷 `init_boot` 只代表注入 KernelSU。需要两者都生效时，应分别修补并刷回对应分区。

完整背景、风险说明和教程见 [vivo/iQOO 适配教程](vivo.md)。

## 文档入口

- [vivo/iQOO 适配教程](vivo.md)
- [英文文档](../README.md)
- [vivo 实现记录](../../DEVLOG-VIVO.md)
- [上游同步注意事项](../../SAKISU-UPSTREAM-SYNC.md)

## 鸣谢

- [ReSukiSU/ReSukiSU](https://github.com/ReSukiSU/ReSukiSU)：当前上游基底。
- [SukiSU-Ultra/SukiSU-Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)：上游血统。
- [KernelSU](https://github.com/tiann/KernelSU)：内核级 root 方案基础。
- 感谢参与 vivo/iQOO 反 root 研究与验证的社区贡献者。

## 许可证

`kernel` 目录遵循 GPL-2.0-only；其余部分按仓库内许可证声明执行。
