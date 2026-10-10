# 版本变更记录

本项目是 [lengran/OpenPrefirePrac](https://github.com/lengran/OpenPrefirePrac) 的兼容性修改版。
下方 **v0.1.47-net10** 为本仓库的改动,**原版历史**见文末。

---

## [v0.1.47-net10] — 2026-10-10

首个兼容版。基于原版 **v0.1.47** 源码,仅做 .NET 10 / 新版 CounterStrikeSharp 兼容性改动,
**未修改任何业务逻辑**。

### 修复

- **修复插件在新版 CounterStrikeSharp 上"加载成功但完全不工作"的问题**
  - 症状:服务器日志显示 `Finished loading plugin Open Prefire Prac`、
    `css_plugins list` 显示 `LOADED`,但游戏内 `!prefire` 无任何反应、
    控制台 `css_prefire` 无输出、插件不产生运行时文件
  - 根因:原版基于 `.NETCoreApp,Version=v8.0` + `CounterStrikeSharp.API 1.0.296` 编译,
    而新版宿主运行时为 `Microsoft.NETCore.App 10.0.3`。
    CLR 能加载程序集但 API 调用静默失败,命令注册与事件订阅全部失效
  - 修复:目标框架升级为 `net10.0`,CounterStrikeSharp.API 升级为 `1.0.376`

### 变更

- `TargetFramework`: `net8.0` → `net10.0`
- `CounterStrikeSharp.API`: `1.0.296` → `1.0.376`
- `Microsoft.Extensions.Localization.Abstractions`: `8.0.3` → `10.0.0`
- 新增 MSBuild 目标 `PruneHostProvidedAssemblies`:编译后自动剔除
  `CounterStrikeSharp.API.dll`、`Microsoft.Extensions.*.dll`、`System.Text.Json.dll`
  等宿主自带程序集,避免部署时版本冲突

### 新增

- **`cfg/prac.cfg`** — 训练服配置,让玩家进服即处于训练状态(等同 MatchZy 练习模式):
  - 训练环境:`sv_cheats 1`、`sv_showimpacts 1`、投掷物轨迹预览(尾迹 6 秒)
  - 无限弹药 + 可带 5 颗投掷物
  - 随处购买、购买时间无限、金钱 9999999
  - 出生免费满甲 + 拆弹器 + AK/M4 + 全套投掷物
  - **死亡立即重生**(`mp_respawn_on_death_ct/t 1` + 重生等待 0)
  - 零冻结时间、回合 60 分钟、999 回合上限
  - `buddha 1` 专注练枪不掉血
  - 队友不碰撞、无人数限制、全服语音
  - `bot_quota 0`(靶子由插件生成)
  - **注意**:需配合 `gamemode_competitive_server.cfg` 才能生效,原因见 README
- **`CHANGELOG.md`** — 本文件

### 已验证环境

| 组件 | 版本 |
|---|---|
| CounterStrikeSharp | 1.0.376 (net10.0) |
| .NET 运行时 | Microsoft.NETCore.App 10.0.3 |
| CS2 专用服务器 | 2000930 |
| 操作系统 | Ubuntu 26.04 |

### 已知限制

- **不支持热重载**:`css_plugins reload OpenPrefirePrac` 会报
  `Could not reload plugin`,修改后需重启服务器实例(CounterStrikeSharp 本身限制)
- 若 CounterStrikeSharp 后续再次升级 .NET 主版本,需按 README 中的方法重新编译
- 原版自 v0.1.47(2025-02-12)后未再更新,本仓库同步跟进官方进展

### 未改动内容

插件业务代码(`OpenPrefirePrac.cs`、`PlayerStatus.cs`、`ServerConfig.cs`、
`ServerStatus.cs`、`Target.cs`、`Translator.cs`)、语言文件(`lang/`)、
训练路线数据(`maps/`)、行为树配置(`resources/bt/`)均与原版 v0.1.47 完全一致。

---

## 原版历史

以下记录来自上游项目 [lengran/OpenPrefirePrac](https://github.com/lengran/OpenPrefirePrac)。

### v0.1.47 — 2025-02-12
- 优化 de_train 上的靶子位置(A 主道 → A 点、梯子房 → A 点)

### v0.1.46 — 2025-01-18
- 修复 "under heaven" 训练路线

### v0.1.45 — 2025-01-13
- 靶子位置调整

### v0.1.44 — 2025-01-09
- 功能更新与修复

### v0.1.43 — 2025-01-07
- 功能更新与修复

### v0.1.42 — 2025-01-03
- 功能更新与修复

### v0.1.41 — 2024-07-27
- 功能更新与修复

### v0.1.40 — 2024-07-19
- 功能更新与修复

> 更早版本请查阅上游仓库的 Releases 页面。
