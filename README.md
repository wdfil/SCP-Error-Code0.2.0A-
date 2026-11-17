# SCP:Error Code服务器查询机器人

基于 QQ 开放平台官方群机器人接口（WebSocket 网关）实现的服务器查询机器人，配合本仓库里的
CarLX 插件 `QQAdminBridge`，可以在 QQ 群里查询 **SCP:Error Code** 服务器状态、击杀榜，
并执行管理员指令。

> 本仓库已做脱敏处理：**不含任何 AppID、AppSecret、令牌、管理员 QQ/OpenID、玩家 UID**。
> 所有敏感值都需要你自己在本地 `config.json` 里填写。

## 目录结构

```
src/
  bot.js                 机器人主程序（单进程，可同时服务多个游戏服）
  config.example.json    配置模板，复制成 config.json 后填写
plugins/
  QQAdminBridge/         CarLX 插件：在游戏服本地开一个 HTTP 管理桥
    QQAdminBridge.cs     给机器人调用的接口（重载RA/刷假人/CASSIE/回合控制等）
```

## 快速开始

1. 安装 Node.js（本仓库只用内置模块，无需 `npm install`）。
2. 复制配置模板：

   ```bash
   cp src/config.example.json src/config.json
   ```

   至少填写：

   | 字段 | 说明 |
   |---|---|
   | `appId` | QQ 开放平台机器人 AppID |
   | `clientSecret` | QQ 开放平台机器人 AppSecret |
   | `adminBridgeToken` | 本地管理桥令牌，必须和插件配置里的 `token` 一致 |
   | `adminQQ` / `adminOpenIds` | 允许执行管理员指令的 QQ / OpenID |
   | `groupOpenIds` | 允许响应的群 OpenID |
   | `leaderboardFile*` | 各服 `killstats` 文件路径 |

3. 编译插件 `QQAdminBridge.cs`，放进游戏服 `CarLX/Plugins/`，并在服配置里同样填好 `token`。
4. 启动机器人：

   ```bash
   node src/bot.js
   ```

## 已实现的指令

**查询类**

| 指令 | 说明 |
|---|---|
| `在线人数` / `/cx` | 各服在线人数 + 手机/PC 合并击杀榜（截图） |
| `玩家名单` | 各服在线玩家名单（截图） |
| `击杀榜` / `一服击杀榜` / `二服击杀榜` / `三服击杀榜` | 指定服务器文字榜 |
| `击杀榜 1\|2\|3` | 按序号选服务器 |
| `PC击杀榜` / `手机击杀榜` | 按平台筛选 |
| `/kd` | 查询本人的等级、经验、所在服、排名、击杀 |
| `/绑定 验证码` | 绑定游戏账号（验证码由游戏内 `.yzd` 生成，5 分钟有效） |
| `/解绑` | 解除绑定 |
| `帮助` | 指令列表 |

**管理员类**（需白名单）

```
重开 / 开回合 / 锁回合 / 解回合
开核 / 关核 / 锁核 / 解核
假人 / 一键刷15假人
播放CASSIE <文本>
```

均在末尾加 `1|2|3` 指定服务器（1=一服，2=二服，3=三服，也可以写服名）。

## 稳定性设计

- **心跳超时自动重连**：超过 2.5 个心跳周期收不到任何网关数据就主动断开重连，
  避免"连接僵死但进程还在"导致机器人不理人。
- **消息处理全兜底**：单条指令抛异常会写日志并尽量回一句提示，不会石沉大海。
- **msg_seq 自增**：同一条群消息下的多条回复使用递增序号，避免被平台判为重复消息而拒收。
- **进程级异常捕获**：`unhandledRejection` / `uncaughtException` 都会落日志。

## 排查

运行时日志写入 `bot.log`，常见关键字：

| 日志 | 含义 |
|---|---|
| `消息处理异常: ...` | 某条指令处理失败（含堆栈） |
| `未处理的异步异常: ...` | 后台异步任务失败 |
| `心跳超时（N秒未收到任何数据），主动重连` | 连接僵死，已自动恢复 |
| `WebSocket 关闭: code=... reason=...` | 网关断开 |
| `图片 multipart 上传失败，尝试 base64` | 平台富媒体接口异常，已自动兜底 |

## 许可

仅供学习与自建服务器使用。SCP:Error Code 及 CarLX 相关版权归各自作者所有。
