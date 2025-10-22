# Stardew Valley 多人联机 Docker Compose 项目

这个项目旨在用最简单的方式自动启动一个 Stardew Valley（星露谷物语）多人联机服务器。

配置步骤与原仓库有些区别，可以参考[使用Docker搭建星露谷最新服务端（Steam下载版）](https://insectmk.cn/posts/b01f80ee/)。

## 重要说明

- 之前的版本会提供游戏文件来创建服务器。为了尊重 ConcernedApe（游戏开发者）的劳动成果并遵守知识产权法律，**将不再提供游戏文件**。用户现在需要使用自己购买的游戏副本。
- 虽然我会尽力发布更新，但没有足够时间进行彻底测试。如果您发现问题，请提交 Issue，我会尽力提供帮助。
- 感谢 printfuck 提供的基础代码。

## 设置指南

### Steam 版本

如果您在 Steam 上拥有这款游戏，此镜像将使用 [steamcmd](https://developer.valvesoftware.com/wiki/SteamCMD)从 Steam 服务器下载游戏。为此，它需要您的 Steam 账户信息。

**凭证环境变量仅在首次构建或更新时需要，游戏运行时不需要。**

```
## 仅在首次构建或更新时设置这些变量
export STEAM_USER=<您的Steam用户名>
export STEAM_PASS=<您的Steam密码>
export STEAM_GUARD=<最新的SteamGuard验证码> # 如果您的账户没有启用Steam Guard保护，则无需设置

docker compose -f docker-compose-steam.yml up
```

#### Steam Guard 验证

如果您的账户受 Steam Guard 保护，构建过程对时间有些敏感。您必须在构建前，打开您的 Steam 手机应用，获取当前的 Steam Guard 验证码，并立即将其设置为 `STEAM_GUARD`环境变量。

**注意：验证码的有效时间比显示的稍长一点，但长不了太多。**

开始构建后，请留意您的手机应用。即使提供了验证码，它可能仍会要求授权，您必须批准该授权。

如果构建失败，或者当您想使用 `docker compose -f docker-compose-steam.yml build --no-cache`更新时，您应该重新设置新的 `STEAM_GUARD`验证码。

```
## 构建完成后，建议移除环境变量
unset STEAM_USER STEAM_PASS STEAM_GUARD
```

### GOG 版本

据我所知，这个过程无法自动化。要使用 GOG 的游戏文件，您需要下载 Linux 安装程序。登录 GOG 账户，进入游戏库，找到 Stardew Valley，将系统切换为 Linux，然后下载游戏安装程序。文件名称类似 `stardew_valley_x.x.x.xxx.sh`。解压此文件（如果在 Windows 上，可以使用 Git Bash），然后将 `data/noarch/`目录内的所有文件复制到 `docker/game_data/`目录下。使用 `docker compose -f docker-compose-gog.yml up`启动容器。更新文件后，使用 `docker compose -f docker-compose-gog.yml build --no-cache`重新构建容器。

### 配置

编辑 `docker-compose.yml`文件，根据您的需求修改配置。设置项的描述比较清晰，说明了它们的作用。

```
environment:
      # VNC 设置
      - VNC_PASSWORD=insecure # VNC 密码
      - DISPLAY_HEIGHT=900    # 显示高度
      - DISPLAY_WIDTH=1200   # 显示宽度
      
      # Always On Server 模组
      ## 禁用这个模组可能就失去了使用本项目的意义？
      - ENABLE_ALWAYSONSERVER_MOD=${ENABLE_ALWAYSONSERVER_MOD-true} # 启用 Always On Server 模组
      - ALWAYS_ON_SERVER_HOTKEY=${ALWAYS_ON_SERVER_HOTKEY-F9} # 服务器模式热键
      - ALWAYS_ON_SERVER_PROFIT_MARGIN=${ALWAYS_ON_SERVER_PROFIT_MARGIN-100} # 利润边际
      - ALWAYS_ON_SERVER_UPGRADE_HOUSE=${ALWAYS_ON_SERVER_UPGRADE_HOUSE-0} # 房屋升级等级
      - ALWAYS_ON_SERVER_PET_NAME=${ALWAYS_ON_SERVER_PET_NAME-Rufus} # 宠物名称
      - ALWAYS_ON_SERVER_FARM_CAVE_CHOICE_MUSHROOMS=${ALWAYS_ON_SERVER_FARM_CAVE_CHOICE_MUSHROOMS-true} # 农场洞穴选择蘑菇
      - ALWAYS_ON_SERVER_COMMUNITY_CENTER_RUN=${ALWAYS_ON_SERVER_COMMUNITY_CENTER_RUN-true} # 社区中心任务开启
      - ALWAYS_ON_SERVER_TIME_OF_DAY_TO_SLEEP=${ALWAYS_ON_SERVER_TIME_OF_DAY_TO_SLEEP-2200} # 每日睡觉时间
      - ALWAYS_ON_SERVER_LOCK_PLAYER_CHESTS=${ALWAYS_ON_SERVER_LOCK_PLAYER_CHESTS-false} # 锁定玩家箱子
      - ALWAYS_ON_SERVER_CLIENTS_CAN_PAUSE=${ALWAYS_ON_SERVER_CLIENTS_CAN_PAUSE-true} # 客户端可以暂停
      - ALWAYS_ON_SERVER_COPY_INVITE_CODE_TO_CLIPBOARD=${ALWAYS_ON_SERVER_COPY_INVITE_CODE_TO_CLIPBOARD-false} # 复制邀请码到剪贴板

      # ... (此处省略了更多节日和超时设置，它们在原文档中列出)
      # 例如：节日开关、各项活动倒计时、各种超时设置等

      # Auto Load Game 模组
      ## 禁用此模组意味着每次启动都需要通过 VNC 手动加载游戏
      - ENABLE_AUTOLOADGAME_MOD=${ENABLE_AUTOLOADGAME-null} # 启用自动加载游戏模组
      - AUTO_LOAD_GAME_LAST_FILE_LOADED=${AUTO_LOAD_GAME_LAST_FILE_LOADED-null} # 自动加载的最后存档文件名
      - AUTO_LOAD_GAME_FORGET_LAST_FILE_ON_TITLE=${AUTO_LOAD_GAME_FORGET_LAST_FILE_ON_TITLE-true} # 在标题界面忘记最后加载的文件
      - AUTO_LOAD_GAME_LOAD_INTO_MULTIPLAYER=${AUTO_LOAD_GAME_LOAD_INTO_MULTIPLAYER-true} # 加载进入多人模式
      
      # Unlimited Players 模组
      - ENABLE_UNLIMITEDPLAYERS_MOD=${ENABLE_UNLIMITEDPLAYERS-true} # 启用无限玩家模组
      - UNLIMITED_PLAYERS_PLAYER_LIMIT=${UNLIMITED_PLAYERS_PLAYER_LIMIT-8} # 玩家数量上限

      # Chat Commands 模组
      - ENABLE_CHATCOMMANDS_MOD=${ENABLE_CHATCOMMANDS_MOD-false} # 启用聊天命令模组

      # Console Commands 模组
      - ENABLE_CONSOLECOMMANDS_MOD=${ENABLE_CONSOLECOMMANDS_MOD-false} # 启用控制台命令模组

      # Time Speed 模组
      - ENABLE_TIMESPEED_MOD=${ENABLE_TIMESPEED_MOD-false} # 启用时间速度模组

      ## 游戏内一天的长度（默认为20小时）
      ##   7.0 = 现实14分钟对应游戏1天（默认）
      ##  10.0 = 20分钟
      ##  15.0 = 30分钟
      ##  20.0 = 40分钟
      ##  30.0 = 1小时
      ## 120.0 = 4小时
      ## 300.0 = 10小时
      ## 600.0 = 20小时（实时）

      - TIME_SPEED_DEFAULT_TICK_LENGTH=${TIME_SPEED_DEFAULT_TICK_LENGTH-7.0} # 默认时间流速
      - TIME_SPEED_TICK_LENGTH_BY_LOCATION_INDOORS=${TIME_SPEED_TICK_LENGTH_BY_LOCATION_INDOORS-7.0} # 室内时间流速
      - TIME_SPEED_TICK_LENGTH_BY_LOCATION_OUTDOORS=${TIME_SPEED_TICK_LENGTH_BY_LOCATION_OUTDOORS-7.0} # 室外时间流速
      - TIME_SPEED_TICK_LENGTH_BY_LOCATION_MINE=${TIME_SPEED_TICK_LENGTH_BY_LOCATION_MINE-7.0} # 矿洞时间流速

      - TIME_SPEED_ENABLE_ON_FESTIVAL_DAYS=${TIME_SPEED_ENABLE_ON_FESTIVAL_DAYS-false} # 节日启用时间控制
      - TIME_SPEED_FREEZE_TIME_AT=${TIME_SPEED_FREEZE_TIME_AT-null} # 在指定时间冻结
      - TIME_SPEED_LOCATION_NOTIFY=${TIME_SPEED_LOCATION_NOTIFY-false} # 地点变更通知

      - TIME_SPEED_KEYS_FREEZE_TIME=${TIME_SPEED_KEYS_FREEZE_TIME-N} # 冻结时间热键
      - TIME_SPEED_KEYS_INCREASE_TICK_INTERVAL=${TIME_SPEED_KEYS_INCREASE_TICK_INTERVAL-OemPeriod} # 加快时间热键
      - TIME_SPEED_KEYS_DECREASE_TICK_INTERVAL=${TIME_SPEED_KEYS_DECREASE_TICK_INTERVAL-OemComma} # 减慢时间热键
      - TIME_SPEED_KEYS_RELOAD_CONFIG=${TIME_SPEED_KEYS_RELOAD_CONFIG-B} # 重载配置热键

      # Crops Anytime Anywhere 模组
      - ENABLE_CROPSANYTIMEANYWHERE_MOD=${ENABLE_CROPSANYTIMEANYWHERE_MOD-false} # 启用随处随时种地模组

      - CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_SPRING=${CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_SPRING-true} # 春季可种植
      - CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_SUMMER=${CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_SUMMER-true} # 夏季可种植
      - CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_FALL=${CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_FALL-true} # 秋季可种植
      - CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_WINTER=${CROPS_ANYTIME_ANYWHERE_ENABLE_IN_SEASONS_WINTER-true} # 冬季可种植

      - CROPS_ANYTIME_ANYWHERE_FARM_ANY_LOCATION=${CROPS_ANYTIME_ANYWHERE_FARM_ANY_LOCATION-true} # 在任何地点耕种

      - CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_DIRT=${CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_DIRT-true} # 强制土地可耕种
      - CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_GRASS=${CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_GRASS-true} # 强制草地可耕种
      - CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_STONE=${CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_STONE-false} # 强制石头地可耕种
      - CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_OTHER=${CROPS_ANYTIME_ANYWHERE_FORCE_TILLABLE_OTHER-false} # 强制其他地形可耕种

      # Friends Forever 模组
      - ENABLE_FRIENDSFOREVER_MOD=${ENABLE_FRIENDSFOREVER_MOD-false} # 启用友谊永固模组

      - FRIENDS_FOREVER_AFFECT_SPOUSE=${FRIENDS_FOREVER_AFFECT_SPOUSE-false} # 影响配偶
      - FRIENDS_FOREVER_AFFECT_DATES=${FRIENDS_FOREVER_AFFECT_DATES-true} # 影响约会对象
      - FRIENDS_FOREVER_AFFECT_EVERYONE_ELSE=${FRIENDS_FOREVER_AFFECT_EVERYONE_ELSE-true} # 影响其他所有人
      - FRIENDS_FOREVER_AFFECT_ANIMALS=${FRIENDS_FOREVER_AFFECT_ANIMALS-true} # 影响动物

      # No Fence Decay 模组
      - ENABLE_NOFENCEDECAY_MOD=${ENABLE_NOFENCEDECAY_MOD-false} # 启用围栏不腐烂模组

      # Non-destructive NPCs 模组
      - ENABLE_NONDESTRUCTIVENPCS_MOD=${ENABLE_NONDESTRUCTIVENPCS_MOD-false} # 启用非破坏性NPC模组
```

## 游戏设置

首次使用时，您需要通过 VNC 或网页界面创建或加载一次游戏存档。之后，每次重启或重建容器时，AutoLoad 模组都会自动跳转到之前加载的游戏存档。AutoLoad 模组的配置文件默认作为卷（volume）挂载，因为它保存了正在进行的游戏存档状态。您也可以将现有的游戏存档复制到 `Saves`卷中，并在环境变量中定义存档名称。启动后，按下 Always On 热键（默认为 F9）即可进入服务器模式。

### VNC 连接

使用 VNC 客户端（如 Windows 上的 `TightVNC`或 Linux 上的 `vncviewer`）连接到服务器。您可以在 `docker-compose.yml`文件中修改 VNC 端口、IP 地址和密码，如下所示：

本地连接示例：

```
# 服务器仅可在本地的 5902 端口访问...
   ports:
     - 127.0.0.1:5902:5900
   # ... 密码是 "insecure"
   environment:
     - VNCPASS=insecure
```

### 网页界面

容器内的 5800 端口（默认映射到主机的 5801 端口）提供了一个网页界面。这比单纯的 VNC 界面更简单、更容易访问。虽然您需要输入 VNC 密码，但不建议将此端口暴露到公网。

## 访问服务器

- **直接 IP 连接**：您需要在互联网上设置直接 IP 访问，通过开放（或转发）端口 24642 来使用"加入局域网游戏"功能。您可以在 compose 文件中随意更改此端口映射。然后其他人就可以通过您的外部 IP"加入局域网游戏"。

（此说明取自模组描述。更多信息请参阅 [Always On Server](https://www.nexusmods.com/stardewvalley/mods/2677?tab=description)）

## 内置模组列表

- [Always On Server](https://www.nexusmods.com/stardewvalley/mods/2677)(默认: 启用，建议保留)
- [Auto Load Game](https://www.nexusmods.com/stardewvalley/mods/2509)(默认: 启用)
- [Crops Anytime Anywhere](https://www.nexusmods.com/stardewvalley/mods/3000)(默认: 禁用)
- [Friends Forever](https://www.nexusmods.com/stardewvalley/mods/1738)(默认: 禁用)
- [No Fence Decay](https://www.nexusmods.com/stardewvalley/mods/1180)(默认: 禁用)
- [Non Destructive NPCs](https://www.nexusmods.com/stardewvalley/mods/5176)(默认: 禁用)
- [Remote Control](https://github.com/Novex/stardew-remote-control)(默认: 启用)
- [TimeSpeed](https://www.nexusmods.com/stardewvalley/mods/169)(默认: 禁用)
- [Unlimited Players](https://www.nexusmods.com/stardewvalley/mods/2213)(默认: 启用)

## 故障排除

### 服务器卡在"等待一天结束"

检查 VNC 界面，确认主机是否卡在了某个提示界面。

### 控制台出现错误信息

通常您可以忽略那里的任何消息。如果游戏无法启动或出现任何错误，您应该查找类似"cannot open display"的消息，这很可能表明存在权限错误。

### VNC 连接问题

通过 VNC 访问游戏，以初始加载或启动一个预先生成的游戏存档。您可以从那里控制服务器，或者编辑 `configs`文件夹中的 `config.json`文件。
