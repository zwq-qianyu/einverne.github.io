---
layout: post
title: "OpenMediaVault 设置"
tagline: ""
description: ""
category:
tags: []
last_updated:
---

OpenMediaVault，是一个开源的基于 Debian Linux 的下一代网络附加存储 (NAS) 解决方案。

## 镜像源


	deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
	# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
	deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
	# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
	deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
	# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
	deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
	# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free

## omv-extras
安装：

    wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash

- <http://omv-extras.org/>


## docker mirror
编辑 `vi /etc/docker/daemon.json`:

	{
		"registry-mirrors": [
			"https://registry.azk8s.cn"
			"https://reg-mirror.qiniu.com",
		],
	  "data-root": "/var/lib/docker"
	}
~
重启：

	/etc/init.d/docker restart

## 开启 sharedfolders
我全新安装的 OpenMediaVault 5.3.4 中，创建共享文件夹，系统不会自动在 sharedfolders 中创建文件夹，查了一下，发现是 OpenMediaVault 在 5.3.3-1 版本中将 sharedfolders 功能给禁用了，官方的[说明](https://github.com/openmediavault/openmediavault/blob/master/deb/openmediavault/debian/changelog#L52) 是可能造成不稳定。不过可以通过如下方法手工开启：

    Disable the '/sharedfolder/<xyz>' feature by default on new
    installations because it makes too much problems.
    It can be enabled by setting the environment variable to
    'OMV_SHAREDFOLDERS_DIR_ENABLED="YES"'. Finally run the command
    'omv-salt stage run prepare' to apply the modified default values
    and 'omv-salt deploy run systemd' to create the unit files.

但是我尝试一下之后发现创建共享文件后，sharedfolder 中依然没有，那我就只能手动 `ln` 了。

	ln -s /srv/dev-disk-by-label-storage/appdata /sharedfolders/appdata
	ln -s /srv/dev-disk-by-label-storage/ruTorrent/ /sharedfolders/ruTorrent

## ruTorrent
Pull 镜像：

	docker pull dockerhub.azk8s.cn/linuxserver/ruTorrent

创建：

```
docker run -d \
  --name=rutorrent \
  -e PUID=1000 \
  -e PGID=1000 \
  -p 8080:80 \
  -p 5000:5000 \
  -p 51415:51413 \
  -p 6881:6881/udp \
  -v /sharedfolders/appdata/ruTorrent:/config \
  -v /sharedfolders/ruTorrent:/downloads \
  --restart unless-stopped \
  dockerhub.azk8s.cn/linuxserver/rutorrent
```

然后根据 [这里](/post/2020/03/rtorrent-and-rutorrent.html) 的说明改一下主题。


