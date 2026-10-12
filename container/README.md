# 容器维护

## RDP 登录与用户

### 同一用户的多个客户端

镜像没有额外限制每个用户只能建立一个 RDP 连接，但 xrdp-sesman 使用默认的
`Policy=Default` 策略：同一个系统用户再次登录时，通常会重新连接该用户已经存在的
活动或断开会话，而不是创建一个独立桌面。因此，多个客户端使用同一个用户名时会看到
同一个桌面，会话也可能在客户端之间切换。

如果需要同时使用多个独立桌面，请为每个客户端创建不同的系统用户。不要让多个用户
共享同一个 RDP 账号。`MaxSessions` 是所有用户合计的会话上限，不是每个用户一个会话。
需要确认当前策略或会话数量限制时，在容器内执行：

```bash
grep -E '^[[:space:]]*#?[[:space:]]*(Policy|MaxSessions)[[:space:]]*=' \
  /etc/xrdp/sesman.ini
```

### 添加用户

先启动容器。服务名是 `application`，RDP 映射到宿主机的 `13389` 端口：

```bash
docker compose up -d
docker compose exec application adduser alice
```

`adduser` 会交互式要求设置密码；使用 `alice` 和该密码登录 RDP。新用户会从
`/etc/skel` 复制 GNOME 会话和输入法的默认文件。如果该用户需要执行管理操作，再将
它加入 `sudo` 组：

```bash
docker compose exec application usermod -aG sudo alice
```

也可以先进入容器再执行相同命令：

```bash
docker compose exec application bash
adduser alice
usermod -aG sudo alice  # 仅在需要 sudo 时执行
```

然后在 RDP 客户端连接 `服务器地址:13389`，使用新用户名登录。SSH 管理端口是
`10022`；也可以通过 SSH 登录容器后执行 `adduser`。

通过上述命令创建的用户只存在于当前容器。`docker compose restart` 或
`stop`/`start` 会保留用户，但 `docker compose down` 删除容器后，重新 `up` 时这些用户
不会保留，只有入口脚本定义的 `ubuntu` 会再次自动创建。需要在容器重建后仍自动拥有
某个用户时，应把用户创建步骤加入自定义 Dockerfile 或入口脚本，并在运行时单独设置
密码；不要把真实密码写入镜像层。

入口脚本会自动创建示例用户 `ubuntu`，其默认密码也是 `ubuntu`。容器暴露到不受信任
的网络前，请立即修改这两个密码。修改当前容器中的密码：

```bash
docker compose exec application passwd ubuntu
docker compose exec application passwd root
```

重新构建镜像时，还应同步修改 [Dockerfile](Dockerfile) 中的示例密码。

## 更新 Google Chrome 安装包

在仓库根目录执行以下命令，删除 `pkg/` 中的旧安装包，再下载最新稳定版。Dockerfile 使用固定文件名 `google-chrome-stable_current_amd64.deb`。

```bash
rm -f container/pkg/google-chrome-stable_current_amd64.deb
wget -O container/pkg/google-chrome-stable_current_amd64.deb \
  https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
dpkg-deb -f container/pkg/google-chrome-stable_current_amd64.deb Version
```

更新安装包后，重新构建镜像：

```bash
docker build -t ubuntu-xrdp:local container
```

## 容器内设置剪贴板同步

镜像构建时在 `/etc/xrdp/xrdp.ini` 中设置了 `cliprdr=false`，因此默认无法在 RDP 客户端和容器桌面之间复制粘贴文本。登录容器后，在容器终端执行：

```bash
sudo sed -i -E 's/^[[:space:]]*cliprdr[[:space:]]*=.*/cliprdr=true/' /etc/xrdp/xrdp.ini
grep -n '^cliprdr=' /etc/xrdp/xrdp.ini
sudo supervisorctl restart xrdp
```

重启 xrdp 会断开当前 RDP 连接。重新连接后，在两端复制新文本进行测试，并确认 RDP 客户端启用了剪贴板重定向。`rdpdr=false` 控制磁盘重定向；仅同步文本无需修改它。

此设置只作用于当前容器。若希望重建容器后仍可同步剪贴板，删除 [Dockerfile](Dockerfile) 中将 `cliprdr=true` 改为 `cliprdr=false` 的命令，再重新构建镜像。

## 容器内设置系统语言为简体中文

镜像已安装简体中文语言包并生成 `zh_CN.UTF-8`。仅修改 `/etc/default/locale` 不够：RDP 启动脚本和终端还会加载 `/usr/local/bin/ubuntu-xrdp_env.bash`，将语言重新设为英语。在容器终端执行：

```bash
locale -a | grep -qi '^zh_CN\.utf8$' || sudo locale-gen zh_CN.UTF-8
sudo update-locale LANG=zh_CN.UTF-8 LANGUAGE=zh_CN:zh
sudo sed -i \
  -e 's/^export LANG=.*/export LANG="zh_CN.UTF-8"/' \
  -e 's/^export LANGUAGE=.*/export LANGUAGE="zh_CN:zh"/' \
  /usr/local/bin/ubuntu-xrdp_env.bash
```

在 GNOME 菜单中**注销**当前用户，然后重新连接 RDP；仅断开连接可能恢复原有会话。重新登录后检查：

```bash
locale | grep -E '^(LANG|LANGUAGE)='
```

预期分别为 `zh_CN.UTF-8` 和 `zh_CN:zh`。这会设置新 RDP 会话及终端的默认语言，时区仍为 `Asia/Hong_Kong`；个别应用可能还有自己的界面语言设置。

此设置只作用于当前容器的登录会话；直接通过 `docker exec` 启动的进程仍可能继承镜像中的英语环境变量。若希望重建后整体默认使用简体中文，还需将 [Dockerfile](Dockerfile) 中的 `update-locale` 和 `ENV LANG`/`LANGUAGE`，以及 [ubuntu-xrdp_env.bash](bin/ubuntu-xrdp_env.bash) 中的语言值同步改为中文，再重新构建镜像。
