# 仓库指南

## 项目用途与目录结构

- 本仓库构建基于 Ubuntu 22.04 的 GNOME 桌面镜像，包含 xrdp、SSH、简繁中文与日语输入支持，以及 Google Chrome。默认语言为美式英语，时区为 `Asia/Hong_Kong`。
- `container/Dockerfile` 定义镜像。构建上下文为 `container/`，因此 `COPY` 路径均相对于该目录。
- `container/bin/entrypoint.bash` 准备用户环境、创建示例用户 `ubuntu` 并启动 Supervisor。`container/bin/ubuntu-xrdp_env.bash` 设置语言环境和时区。
- `container/supervisor/*.conf` 启动 D-Bus、logind、sshd、xrdp-sesman 和 xrdp。`container/xrdp/*.sh` 是 xrdp 会话脚本。
- `container/pkg/google-chrome-stable_current_amd64.deb` 是已提交到仓库的构建文件；如果移动该文件，需同步调整 `COPY` 路径和安装步骤。
- `docker-compose.yml` 从 `container/` 构建镜像，将 RDP 映射到宿主机的 13389 端口、SSH 映射到 10022 端口，并为 Chrome 配置 `SYS_ADMIN` 权限及 2 GiB 共享内存。

## 修改镜像

- 在 `container/Dockerfile` 中维护软件包安装及镜像配置；在 `container/bin/` 中维护进程启动逻辑；在 `container/supervisor/` 中维护 Supervisor 配置。
- root 的 SSH 和 RDP 登录必须保持禁用，root 密码在构建时和每次容器启动时锁定。日常使用 `ubuntu`，管理操作通过 sudo 或宿主机 Docker 执行。入口脚本仍为示例账户设置 `ubuntu/ubuntu` 默认凭据；重建容器后必须在开放公网访问前重新设置强密码，不要把真实密码写入镜像层。
- 修改桌面会话启动流程时，同时检查 `container/xrdp/startwm.sh` 和 `entrypoint.bash` 写入 `/etc/skel/.xsessionrc` 的环境变量。
- GNOME 输入源默认值位于 `container/dconf/00-input-sources`；简体使用 IBus Libpinyin，繁体使用 Rime 的 `luna_pinyin_tw` 方案，其配置位于 `container/rime/default.custom.yaml`。修改输入法时，同步检查 Dockerfile 中的软件包和语言环境。
- GitHub Actions 工作流在推送到 `main` 时发布镜像。修改镜像标签或构建上下文时，同步更新 Compose 配置和文档。

## 验证

- 修改 Compose 配置后运行 `docker compose config`。
- 修改 Dockerfile 或构建文件后运行 `docker compose build`。
- 修改运行时行为后，运行 `docker compose up -d`，查看 `docker compose logs`，并检查宿主机 10022 端口的 SSH 和 13389 端口的 RDP。测试结束后运行 `docker compose down`。
- 修改语言或输入源时，在新用户的 GNOME 会话中检查美式英语界面、香港时区，以及英语、日语、简体拼音、繁体拼音输入源。
- 本仓库没有自动化测试套件。如果环境中没有 Docker，应说明未执行构建和运行时验证。
