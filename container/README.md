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
