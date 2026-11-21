# 应用包名迁移（遗留待办 · C 项）

> **状态：暂缓，本文件只记录方案，未执行。** 执行前需先确定新包名（见文末建议）。

## 现状

- 当前应用包名 / applicationId：`com.yangyang2857rgb.onestar`
- 已在 v0.10.0（2026-11-08）换过一次（见 `CHANGELOG.md`、`docs/version-timeline.md`），本次为第二次
- 源码目录：`java/com/yangyang2857rgb/onestar/`、`test/com/yangyang2857rgb/onestar/`

## 为什么暂缓

Android 应用包名 = 应用身份，改包名**等于换一个新应用**：

- 旧应用需卸载重装，数据默认不迁移（除非自写迁移逻辑）
- 桌面小组件（`StarWidget`）失效，需重新添加
- 系统、备份、应用商店识别全部变化

对仍在 0.x、走侧载分发（GitHub Release）的应用来说，用户量小、代价可控，但**没有紧急理由**，故暂缓。

## 迁移范围

| 类型 | 位置 |
|---|---|
| manifest | `AndroidManifest.xml` 的 `package` |
| 源码 package | `java/com/yangyang2857rgb/onestar/*.java`（约 21 个）+ `test/` |
| 目录 | `java/`、`test/` 下 `com/yangyang2857rgb/onestar` → 新路径 |
| 文档 | `README.md`、`README.zh-CN.md`、`docs/release-spec.md`、`docs/signing-and-upgrade.md`、`docs/version-timeline.md`、`docs/cross-platform-release-plan.md`、`CHANGELOG.md` |

## 新包名建议（待确认）

| 项目 | 旧 | 建议新 |
|---|---|---|
| 应用包名 | `com.yangyang2857rgb.onestar` | `com.wangsunyang.onestar` |

## 执行时机建议

- 与 appfunction-guard 的包名迁移**一起做**，一次性清掉所有 `yangyang2857` 残留。
- 或等有更强理由（上架应用商店、需要正式身份、首次大规模分发）时再做。
