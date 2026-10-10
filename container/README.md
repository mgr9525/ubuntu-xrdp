# 容器维护

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
