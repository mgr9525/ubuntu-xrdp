# Telegram

通过 RDP 以 `ubuntu` 用户登录后，在**容器内**的终端执行以下命令。使用 Telegram 官方 Linux x64 压缩包，安装到当前用户的 `~/.local/opt/Telegram`，不需要 Flatpak 或 Snap。

## 安装

先安装下载、解压和桌面快捷方式所需的工具：

```bash
sudo apt update && sudo apt install -y wget xz-utils xdg-user-dirs libglib2.0-bin
```

官方链接会指向当前稳定版。下载并校验压缩包后，解压到用户目录，创建启动命令：

```bash
wget -O /tmp/telegram-linux-x64.tar.xz https://telegram.org/dl/desktop/linux &&
tar -tJf /tmp/telegram-linux-x64.tar.xz >/dev/null &&
mkdir -p "$HOME/.local/opt" "$HOME/.local/bin" &&
tar -xJf /tmp/telegram-linux-x64.tar.xz -C "$HOME/.local/opt" &&
ln -sfn "$HOME/.local/opt/Telegram/Telegram" "$HOME/.local/bin/telegram-desktop"
```

在 RDP 桌面中启动：

```bash
"$HOME/.local/bin/telegram-desktop"
```

安装命令只创建了终端启动命令。若要在 GNOME 应用菜单和桌面上添加快捷方式，继续在同一个 RDP 终端执行：

```bash
desktop_dir="$(xdg-user-dir DESKTOP)"
launcher_dir="$HOME/.local/share/applications"
icon_dir="$HOME/.local/share/icons"
mkdir -p "$desktop_dir" "$launcher_dir" "$icon_dir"
wget -O "$icon_dir/telegram.png" https://telegram.org/img/t_logo.png
cat > "$launcher_dir/telegram.desktop" <<EOF
[Desktop Entry]
Type=Application
Name=Telegram
Exec=$HOME/.local/bin/telegram-desktop
Icon=$icon_dir/telegram.png
Terminal=false
Categories=Network;InstantMessaging;
StartupNotify=true
EOF
install -m 755 "$launcher_dir/telegram.desktop" "$desktop_dir/Telegram.desktop"
gio set "$desktop_dir/Telegram.desktop" metadata::trusted true
```

若桌面图标仍提示不受信任，右键点击图标并选择“允许启动”。

## 更新

先在 Telegram 菜单中退出程序，再运行：

```bash
wget -O /tmp/telegram-linux-x64.tar.xz https://telegram.org/dl/desktop/linux &&
tar -tJf /tmp/telegram-linux-x64.tar.xz >/dev/null &&
tar -xJf /tmp/telegram-linux-x64.tar.xz -C "$HOME/.local/opt" &&
"$HOME/.local/bin/telegram-desktop"
```

更新只覆盖程序文件，不删除 `Telegram` 目录中的用户数据。软件和登录数据都位于容器内；重新创建容器前，应备份或持久化用户主目录。
