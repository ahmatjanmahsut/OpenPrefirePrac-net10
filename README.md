# OpenPrefirePrac — CounterStrikeSharp net10 兼容版

> **这是 [lengran/OpenPrefirePrac](https://github.com/lengran/OpenPrefirePrac) 的兼容性修改版**。
> 原项目基于 **.NET 8 + CounterStrikeSharp 1.0.296** 编译,而新版 CounterStrikeSharp(1.0.3xx 起)
> 已升级到 **.NET 10**,导致原版插件在新宿主上**加载成功但完全不工作**。
> 本仓库在原版 v0.1.47 源码基础上重新编译为 net10,并补充了可直接使用的训练服配置。

![License](https://img.shields.io/badge/license-MIT-blue)
![Framework](https://img.shields.io/badge/.NET-10.0-purple)
![CSS](https://img.shields.io/badge/CounterStrikeSharp-1.0.376-green)

---

## 目录

- [为什么需要这个版本](#为什么需要这个版本)
- [与原版的差异](#与原版的差异)
- [功能特性](#功能特性)
- [编译方法](#编译方法)
- [部署方法](#部署方法)
- [游戏内使用](#游戏内使用)
- [训练服默认配置(推荐)](#训练服默认配置推荐)
- [目录结构](#目录结构)
- [常见问题](#常见问题)
- [版本变更](./CHANGELOG.md)
- [致谢与许可](#致谢与许可)

---

## 为什么需要这个版本

### 症状(极具迷惑性)

插件在服务器上表现为"**已加载但不工作**":

\`\`\`
[#2:LOADED]: "Open Prefire Prac" (0.1.47) by Lengran     ← 显示已加载
Finished loading plugin Open Prefire Prac                 ← 日志无报错
\`\`\`

但实际上:

- 聊天框输入 `!prefire` **毫无反应**,菜单不弹出
- 控制台执行 `css_prefire` **没有任何输出**
- 插件目录不产生任何运行时文件

### 根因

用 `strings` 查看插件 DLL 的目标框架即可发现问题:

\`\`\`bash
strings OpenPrefirePrac.dll | grep -oE '\.NETCoreApp,Version=v[0-9.]+'
# 原版输出: .NETCoreApp,Version=v8.0     ← 旧框架
\`\`\`

而新版 CounterStrikeSharp 宿主运行时是:

\`\`\`bash
addons/counterstrikesharp/dotnet/dotnet --list-runtimes
# Microsoft.NETCore.App 10.0.3            ← 宿主是 .NET 10
\`\`\`

**.NET 8 编译的插件在 .NET 10 宿主里能被 CLR 加载(所以显示 LOADED),但调用
CounterStrikeSharp.API 时会因版本不匹配而静默失败** —— 命令注册、事件订阅全部失效。

### 判断方法(通用技巧)

任何 CSS 插件出现"加载了但没反应",都可以这样排查:

\`\`\`bash
# 1. 看插件的目标框架
strings plugins/某插件/某插件.dll | grep -oE '\.NETCoreApp,Version=v[0-9.]+'

# 2. 看宿主运行时
addons/counterstrikesharp/dotnet/dotnet --list-runtimes | grep NETCore

# 3. 两者主版本不一致 → 就是这个原因
\`\`\`

---

## 与原版的差异

本仓库相对[原项目](https://github.com/lengran/OpenPrefirePrac) v0.1.47 **只做了兼容性改动,
没有修改任何业务逻辑**:

| 文件 | 改动 |
|---|---|
| `OpenPrefirePrac.csproj` | `TargetFramework`: net8.0 → **net10.0** |
| `OpenPrefirePrac.csproj` | `CounterStrikeSharp.API`: 1.0.296 → **1.0.376**(与新版宿主一致) |
| `OpenPrefirePrac.csproj` | `Microsoft.Extensions.Localization.Abstractions`: 8.0.3 → **10.0.0** |
| `OpenPrefirePrac.csproj` | 新增 `PruneHostProvidedAssemblies` 目标:编译后自动剔除 CSS 宿主自带的 DLL,避免版本冲突 |
| `cfg/prac.cfg`(新增) | 训练服配置,进服即训练状态(详见后文) |
| `CHANGELOG.md`(新增) | 版本变更记录 |

插件源码(`*.cs`)、语言文件、训练路线数据(`maps/`)、行为树配置(`resources/bt/`)均**未改动**。

---

## 功能特性

沿用原版能力:

- **预瞄路线训练**:内置 9 张竞技图(de_dust2 / de_mirage / de_inferno / de_nuke /
  de_overpass / de_ancient / de_anubis / de_train / de_vertigo)的数十条预瞄路线
- **多人同时训练**:不同玩家可各自训练互不干扰
- **靶子(Bot)生成**:按路线自动生成站位靶子,支持随机 / 全部两种模式
- **难度分级**:6 档难度(不同回血策略)
- **Bot 武器可选**:随机 / UMP45 / AK47 / Scout / AWP
- **Bot 瞄准模式**:原生行为 / CSS 锁定爆头 / 行为树困难模式
- **多语言**:内置英文、葡萄牙语、简体中文(`!prefire lang zh` 切换)
- **进度记录**:记录每条路线的完成情况与最佳成绩

---

## 编译方法

### 前置要求

- [.NET SDK 10.0](https://dotnet.microsoft.com/download) 或更高
- 网络可访问 NuGet(需要还原 CounterStrikeSharp.API 等包)

### 编译

\`\`\`bash
git clone https://github.com/<你的用户名>/OpenPrefirePrac.git
cd OpenPrefirePrac
dotnet build -c Release
\`\`\`

产物在 `bin/Release/net10.0/`。

### 需要部署的文件

只部署**插件自身与它的专属依赖**:

\`\`\`
OpenPrefirePrac.dll
OpenPrefirePrac.deps.json          (可选,建议一并带上)
MaxMind.Db.dll
MaxMind.GeoIP2.dll
McMaster.NETCore.Plugins.dll
Microsoft.DotNet.PlatformAbstractions.dll
lang/  maps/  resources/           (数据目录,必须保留)
\`\`\`

> ⚠️ **不要**把 `Serilog*.dll`、`FastGenericNew.dll`、`Scrutor.dll`、`Tomlyn.dll`、
> `System.Diagnostics.EventLog.dll`、`CounterStrikeSharp.API.dll` 复制进插件目录 ——
> 这些由 CounterStrikeSharp 宿主提供,重复分发会导致版本冲突。
> 仓库中的 `PruneHostProvidedAssemblies` 目标已自动处理。

---

## 部署方法

1. 把上一步的文件放进:

\`\`\`
game/csgo/addons/counterstrikesharp/plugins/OpenPrefirePrac/
\`\`\`

2. 可选:把 `default_cfg.json.example` 复制为 `default_cfg.json` 并调整默认值

3. **重启服务器**(注意:不能用 `css_plugins reload`,该类插件会报
   `Could not reload plugin`,必须重启实例)

4. 验证:

\`\`\`bash
css_plugins list        # 应看到 [#N:LOADED] "Open Prefire Prac"
\`\`\`

---

## 游戏内使用

| 操作 | 命令 |
|---|---|
| 打开训练主菜单 | `!prefire` 或控制台 `css_prefire` |
| 开始指定路线 | `!prefire prac <编号>` |
| 更换地图 | `!prefire map de_mirage` |
| 设置难度(1-6) | `!prefire df 3` |
| 训练模式 | `!prefire mode rand` / `!prefire mode full` |
| 靶子武器 | `!prefire bw ak` / `ump` / `sct` / `awp` / `rand` |
| 切换语言 | `!prefire lang zh` |
| 退出训练 | `!prefire exit` |
| 查看帮助 | `!prefire help` |

### 推荐启动参数

原项目作者建议训练服使用:

\`\`\`bash
./cs2 -dedicated -port 27025 +map de_dust2 -maxplayers 32 \\
     -maxplayers_override 64 +game_alias competitive +sv_hibernate_when_empty 0
\`\`\`

> `-maxplayers_override 64` **最重要** —— 它允许单边超过 5 个 bot,是多人同时训练的前提。

---

## 训练服默认配置(推荐)

仓库附带的 `cfg/prac.cfg` 让玩家**进服即处于训练状态**(效果等同 MatchZy 的练习模式):

| 类别 | 设置 |
|---|---|
| 调试 | `sv_cheats 1`、`sv_showimpacts 1`、投掷物轨迹预览(尾迹 6 秒) |
| 弹药 | `sv_infinite_ammo 1`、可带 5 颗投掷物 |
| 购买 | 随处可买、时间无限、金钱 9999999 |
| 出生装备 | 免费满甲 + 拆弹器 + AK/M4 + 全套投掷物 |
| 重生 | **死亡立即重生** |
| 回合 | 零冻结、回合 60 分钟、999 回合上限 |
| 免伤 | `buddha 1`(专注练枪不掉血) |
| 团队 | 队友不碰撞、无人数限制、全服语音 |
| Bot | `bot_quota 0`(靶子交给插件生成) |

### ⚠️ 关键:配置文件要放对位置

CS2 的配置加载顺序是:

\`\`\`
server.cfg  →  gamemode_competitive.cfg  →  gamemode_competitive_server.cfg
\`\`\`

**`gamemode_competitive.cfg` 会把 `mp_freezetime`、`mp_roundtime`、`mp_buy_anywhere`、
`mp_respawn_on_death_*` 等 convar 打回竞技默认值**。所以:

- ❌ 只写 `server.cfg` → 重启后训练设置失效(实测 `mp_freezetime` 变回 15)
- ✅ **必须在 `gamemode_competitive_server.cfg` 里 exec**,它在 gamemode 配置之后执行

部署方式(三层):

\`\`\`bash
game/csgo/cfg/prac.cfg                          # 训练 convar 主体
game/csgo/cfg/server.cfg                        # 内容: exec prac.cfg
game/csgo/cfg/gamemode_competitive_server.cfg   # 内容: exec prac.cfg   ← 关键
\`\`\`

启动日志中应能看到:

\`\`\`
execing gamemode_competitive.cfg
execing gamemode_competitive_server.cfg
execing prac.cfg
\`\`\`

> **CS2 已移除的 convar**(写入会报 `Unknown command`):
> `mp_damage_print`、`mp_death_drop_breachcharge`、`mp_grenade_limit_total`

---

## 目录结构

\`\`\`
.
├── OpenPrefirePrac.cs          插件主逻辑(未改动业务代码)
├── PlayerStatus.cs             玩家训练状态
├── ServerConfig.cs             服务器配置
├── ServerStatus.cs             服务器状态
├── Target.cs                   靶子(Bot)管理与生成
├── Translator.cs               多语言支持
├── OpenPrefirePrac.csproj      工程文件(已改为 net10.0)
├── default_cfg.json.example    默认配置模板
├── calculate_height.py         辅助脚本:转换游戏内 getpos 输出为靶子高度
├── lang/                       语言文件(en / pt-BR / zh)
├── maps/                       训练路线数据(9 张图)
├── resources/bt/               行为树配置(困难模式 Bot AI)
├── resources/GeoLite2-Country.mmdb   GeoIP 数据库(语言自动选择用)
├── cfg/prac.cfg                训练服配置(本项目新增)
├── CHANGELOG.md                版本变更(本项目新增)
└── LICENSE                     MIT(保留原作者署名)
\`\`\`

---

## 常见问题

### Q:插件显示 LOADED 但 `!prefire` 没反应?

检查目标框架是否与宿主一致(见 [为什么需要这个版本](#为什么需要这个版本))。
如果宿主又升级了 .NET 版本,需要按同样方式重新编译。

### Q:重启后训练设置失效了?

配置文件位置不对。必须放在 `gamemode_<模式>_server.cfg` 里(见上文)。

### Q:`css_plugins reload OpenPrefirePrac` 报错?

这是 CounterStrikeSharp 本身的限制,该类插件不支持热重载,**必须重启服务器实例**。

### Q:多人能同时训练吗?

可以。需要启动参数加 `-maxplayers_override 64`,否则单边 bot 数量受限。

### Q:靶子不生成 / 路线列表是空的?

确认插件目录下的 `maps/` 与 `resources/` 完整保留。这两个目录是数据,不能漏传。

---

## 致谢与许可

- **原作者**:[lengran](https://github.com/lengran) —— 本项目 99% 的工作来自
  [OpenPrefirePrac](https://github.com/lengran/OpenPrefirePrac),本仓库仅做 net10 兼容性重新编译
- **依赖**:[CounterStrikeSharp](https://github.com/roflmuffin/CounterStrikeSharp)、
  [MaxMind GeoIP2](https://github.com/maxmind/GeoIP2-dotnet)、
  [McMaster.NETCore.Plugins](https://github.com/natemcmaster/DotNetCorePlugins)

**许可证:MIT**(见 [LICENSE](./LICENSE))。原版权声明 `Copyright (c) 2024 lengran` 已完整保留。

本项目同样以 MIT 协议开源,欢迎自由使用与修改。
