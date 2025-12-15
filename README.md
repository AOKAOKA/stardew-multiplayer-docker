# Stardew Valley 多人联机 Docker Compose 项目

这个项目旨在用最简单的方式自动启动一个 Stardew Valley（星露谷物语）多人联机服务器。

## 重要说明

此仓库基于[insectmk/stardew-multiplayer-docker](https://github.com/insectmk/stardew-multiplayer-docker)优化改造。

## 设置指南

此仓库目前只用于个人记录更改使用！

在开始之前默认服务器存在`docker`、`docker-compose`环境。

### Steam 版本

### 拉取仓库

```bash
# 使用Github源仓库
git clone https://github.com/insectmk/stardew-multiplayer-docker.git
```

### Steam登录配置

你必须要在Steam上购买星露谷，构建镜像时，会使用[steamcmd](https://developer.valvesoftware.com/wiki/SteamCMD)从Steam下载最新的星露谷。

需要配置Steam账户环境变量（镜像构建完成后就不需要了），修改`.env`文件，编辑以下内容

```properties
## 仅在首次构建或更新时设置这些变量
STEAM_USER=<你的Steam用户名>
STEAM_PASS=<你的Steam密码>
STEAM_GUARD=<最新的Steam令牌验证码> # 如果你的账户没有启用Steam令牌保护，则无需设置
```

**注意：如果你使用了Steam令牌验证码的话，构建编排的时候要及时修改`.env`的`STEAM_GUARD`为最新令牌**

### SMAPI插件平台

由于原仓库构建镜像时，SMAPI在构建镜像的时候从Github下载的，拖慢了构建镜像的速度，现在改成了可预先下载，使用本地SMAPI构建。

到[SMAPI-Releases页面](https://github.com/Pathoschild/SMAPI/releases)下载你需要的插件平台版本，如：`SMAPI-4.3.2-installer.zip`，并放入到`docker/smapi`文件夹（需要新建）中。

（PS：之前在某个文章的评论区看到，[ALOS模组](https://www.nexusmods.com/stardewvalley/mods/2677)，在每日结算的时候没有自动确认，导致其他玩家一直在等待，需要手动点击服主机器人的OK，解决方案是将SMAPI降级到`4.1.7`以前的版本，这里使用的本项目原作者的版本->[4.1.10](https://github.com/Pathoschild/SMAPI/releases/tag/4.1.10)，经测试一样有效。）

放入后，修改`docker/Dockerfile-steam`文件：

```dockerfile
# 修改这行，将SMAPI-4.1.10-installer.zip改为你对应的SMAPI Installer包
COPY smapi/SMAPI-4.3.2-installer.zip /data/nexus.zip
```

### VNC密码

编辑`docker-compose-steam.yml`，修改NOVNC Web密码：

```yaml
services:
  valley:
    environment:
      # VNC
      - VNC_PASSWORD=<你的密码>
```

### 创建Docker编排

回到项目根目录，使用以下指令创建编排

```bash
sudo docker compose -f docker-compose-steam.yml up
```

- 编排会先创建镜像（下载各种环境以及游戏本体），请耐心等待，我的服务器构建花费了40分钟左右。
- 如果到Steam下载游戏时，令牌验证码已经失效，更新`.env`中的`STEAM_GUARD`为最新验证码后，重新使用上面的指令创建编排。

### 开服

创建编排后，会在项目根目录生成一个`valley_saves`文件夹，这个文件夹就是你的存档目录，你可以将你的存档上传到这个目录下。

Windows存档地址`C:\Users\你的用户名\AppData\Roaming\StardewValley\Saves`，可以看到有很多文件夹，文件夹名称对应的就是你的农场名称。

### 开放端口

容器使用了`5801`和`24642`端口这两个端口，需要在云服务器控制台的安全组开放，防火墙也放行一下。

- `5801`：NoVNC 的 Web端，不用下载VNC客户端，直接访问`服务IP:5801`就能访问星露谷桌面了
- `24642`：服务端端口，星露谷默认端口，在星露谷客户端点局域网连接的时候不用带上这个端口，直接给IP或者域名就行了

#### 进入VNC Web

默认VNC Web端开放在`5801`端口，在浏览器访问`你的服务器IP:5801`就能看到游戏页面了，跟你在电脑上玩的星露谷的UI一样。

## 模组

### 内置模组列表

- [Always On Server](https://www.nexusmods.com/stardewvalley/mods/2677)(默认: 启用，建议保留)
- [Auto Load Game](https://www.nexusmods.com/stardewvalley/mods/2509)(默认: 启用)
- [Crops Anytime Anywhere](https://www.nexusmods.com/stardewvalley/mods/3000)(默认: 禁用)
- [Friends Forever](https://www.nexusmods.com/stardewvalley/mods/1738)(默认: 禁用)
- [No Fence Decay](https://www.nexusmods.com/stardewvalley/mods/1180)(默认: 禁用)
- [Non Destructive NPCs](https://www.nexusmods.com/stardewvalley/mods/5176)(默认: 禁用)
- [Remote Control](https://github.com/Novex/stardew-remote-control)(默认: 启用)
- [TimeSpeed](https://www.nexusmods.com/stardewvalley/mods/169)(默认: 禁用)
- [Unlimited Players](https://www.nexusmods.com/stardewvalley/mods/2213)(默认: 启用)
- [Tractor 拖拉机模组](https://www.nexusmods.com/stardewvalley/mods/1401)(默认：启用)：客户端也需安装

### 配置

编辑 `docker-compose-steam.yml`文件，根据需求修改配置。

大部分的配置都使用了中文注释，参考注释进行配置。

## 注意事项

### 服务器卡在"等待一天结束"

检查 VNC 界面，确认主机是否卡在了某个提示界面（每天结束后，需要点一下结算确认）。

### 控制台出现错误信息

只要能正常游玩，报错不用管。

但是如果游戏无法启动或出现任何错误，找到类似"cannot open display"的消息，表示可能存在权限错误。
