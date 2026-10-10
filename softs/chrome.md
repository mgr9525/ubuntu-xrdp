# Google Chrome

通过 RDP 以 `ubuntu` 用户登录，在**容器内**的终端执行。更新前先退出 Chrome。

## 更新

镜像安装的 Chrome `.deb` 通常会配置 Google 的 APT 软件源。先尝试从软件源更新：

```bash
sudo apt update &&
sudo apt install --only-upgrade -y google-chrome-stable &&
google-chrome --version
```

若软件源中没有新版，直接下载并安装 Google 提供的最新稳定版 `.deb`：

```bash
wget -O /tmp/google-chrome-stable_current_amd64.deb \
  https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb &&
sudo apt install -y /tmp/google-chrome-stable_current_amd64.deb &&
google-chrome --version
```

更新会保留当前用户的 Chrome 配置。以上操作只影响当前容器；重新创建容器仍会使用镜像中的版本。若要更新镜像内的安装包，参见 [container/README.md](../container/README.md)。

如果启动时仍出现 `Failed to move to new namespace`，需要检查容器权限；更新 Chrome 不会自动解决这个问题。
