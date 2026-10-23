FROM ubuntu:22.04

ENV TZ=Asia/Hong_Kong

RUN apt update && DEBIAN_FRONTEND=noninteractive apt -y install \
  bash-completion \
  build-essential \
  curl \
  dbus-x11 \
  dconf-cli \
  fonts-noto \
  fonts-noto-cjk \
  git \
  ibus-libpinyin \
  ibus-mozc \
  ibus-rime \
  language-pack-en \
  language-pack-ja-base \
  language-pack-ja \
  language-pack-zh-hans \
  language-pack-zh-hant \
  less \
  locales \
  nano \
  net-tools \
  openssh-server \
  rime-data-luna-pinyin \
  sudo \
  supervisor \
  tzdata \
  ubuntu-gnome-desktop \
  vim \
  xrdp \
  && locale-gen en_US.UTF-8 zh_CN.UTF-8 zh_TW.UTF-8 ja_JP.UTF-8 \
  && update-locale LANG=en_US.UTF-8 LANGUAGE=en_US:en \
  && ln -snf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime \
  && echo Asia/Hong_Kong > /etc/timezone \
  && apt clean && rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en

COPY dconf/00-input-sources /etc/dconf/db/local.d/00-input-sources
RUN mkdir -p /etc/dconf/profile \
  && if [ ! -e /etc/dconf/profile/user ]; then printf 'user-db:user\n' > /etc/dconf/profile/user; fi \
  && if ! grep -qx 'system-db:local' /etc/dconf/profile/user; then sed -i '/^user-db:/a system-db:local' /etc/dconf/profile/user; fi \
  && dconf update

COPY rime/default.custom.yaml /etc/skel/.config/ibus/rime/default.custom.yaml
COPY rime/default.custom.yaml /root/.config/ibus/rime/default.custom.yaml

# sshd
RUN mkdir -p /var/run/sshd \
  # Put these before Include directives: sshd uses the first value it reads.
  && sed -i -E '/^[[:space:]]*(PermitRootLogin|PasswordAuthentication)[[:space:]]/Id' /etc/ssh/sshd_config \
  && sed -i '1iPermitRootLogin no\nPasswordAuthentication yes' /etc/ssh/sshd_config \
  && passwd -l root \
  && /usr/sbin/sshd -t \
  && /usr/sbin/sshd -T | grep -qx 'permitrootlogin no' \
  && /usr/sbin/sshd -T | grep -qx 'passwordauthentication yes'


# xrdp
RUN mkdir -p /var/run/dbus \
  && adduser xrdp ssl-cert \
  && apt -y remove gnome-initial-setup \
  && sed -i -E 's/^rdpdr=true/rdpdr=false/g' /etc/xrdp/xrdp.ini \
  && sed -i -E 's/^cliprdr=true/cliprdr=false/g' /etc/xrdp/xrdp.ini \
  && sed -i -E '/^[[:space:]]*\[Security\][[:space:]]*$/,/^[[:space:]]*\[[^]]+\][[:space:]]*$/ s/^[[:space:]]*AllowRootLogin[[:space:]]*=.*/AllowRootLogin=false/' /etc/xrdp/sesman.ini \
  && awk '/^[[:space:]]*\[/ { security = ($0 ~ /^[[:space:]]*\[Security\][[:space:]]*$/) } security && /^[[:space:]]*AllowRootLogin[[:space:]]*=/ { found = 1; if ($0 !~ /^AllowRootLogin=false$/) invalid = 1 } END { exit (!found || invalid) }' /etc/xrdp/sesman.ini

COPY xrdp/*.sh /etc/xrdp/
RUN chmod +x /etc/xrdp/*.sh

COPY pkg/google-chrome-stable_current_amd64.deb /usr/local/share/
RUN dpkg -i /usr/local/share/google-chrome-stable_current_amd64.deb


# supervisor
COPY bin/* /usr/local/bin/
RUN chmod +x /usr/local/bin/*
COPY supervisor/* /etc/supervisor/conf.d/
ENTRYPOINT ["entrypoint.bash"]
