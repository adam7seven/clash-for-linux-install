# Linux 一键安装 Clash

因为有在服务器上使用代理的需求，试过很多开源脚本，总是遇到各种问题。于是自己动手，丰衣足食。对安装过程及后续使用进行了友好封装，部署使用起来优雅、简单、明确。

基于 `Clash` 项目作者删库前最新的 `Premium` 版本，可自行更换其他内核版本。

文末[引用](#引用)中收集了各种内核和 `GUI` 客户端版本的下载地址。

![preview](resource/preview.png)

## 环境要求

- 需要 `root` 或 `sudo` 权限。
- 具备 `bash` 和 `systemd` 的系统环境。
- 已适配：`CentOS 7.x`、`Debian 12.x`、`Ubuntu 24.x`。

## 开始使用

### 一键安装

> [!NOTE]
>
> [常见问题](#常见问题)

```bash
git clone https://github.com/adam7seven/clash-for-linux-install.git && cd clash-for-linux-install && sudo bash -c '. install.sh; exec bash'
```

- 上述脚本已使用[代理加速下载](https://ghproxy.link/)，如克隆失败请自行更换。
- ~~不懂什么是订阅链接的小白可参考~~：[issue#1](https://github.com/nelvko/clash-for-linux-install/issues/1)
- 没有订阅？[click me](https://次元.net/auth/register?code=oUbI)

### 命令一览

```bash
$ clash
Usage:                                    
    clashon                开启代理       
    clashoff               关闭代理       
    clashui                面板地址       
    clashtun [on|off]      Tun模式        
    clashsecret [secret]   查看/设置密钥  
    clashmixin [-e|-r]     Mixin配置      
    clashupdate [auto|log] 更新订阅
```

### 基础使用

```bash
$ clashoff
😼 已关闭代理环境

$ clashon
😼 已开启代理环境

$ clashui
😼 Web 面板地址...
```

- 使用 `systemctl` 控制 `clash` 启停后，还需调整代理环境变量的值（http_proxy 等）。因为应用程序在发起网络请求时，会通过其指定的代理转发流量，不调整会造成关闭代理后仍转发导致请求失败，开启代理同理。
- 以上命令集成了上述流程。

### 定时更新配置

```bash
$ clashupdate [url]
😼 配置更新成功，已重启生效

$ clashupdate auto [url]
😼 定时任务设置成功

$ clashupdate log
✅ 2024-12-13 23:38:56 配置更新成功 ...
```

- 不指定 `url` 默认使用安装时填的订阅。
- 通过粘贴配置内容安装的，更新配置步骤：[pr#24](https://github.com/nelvko/clash-for-linux-install/pull/24#issuecomment-2565054701)
- 可通过 `crontab -e` 修改更新频率及订阅链接。
- 依赖 [`yq`](https://github.com/mikefarah/yq/releases) 命令实现 [`Mixin`](#mixin-配置)，如下载失败请自行安装到 `PATH` 路径内。

### Web 控制台设置密钥（推荐）

```bash
$ clashsecret xxx
😼 密钥更新成功，已重启生效

$ clashsecret
😼 当前密钥：xxx
```

### `Tun` 模式

```bash
$ clashtun
😼 Tun 状态：关闭

$ clashtun on
😼 Tun 模式已开启
```

- 作用：实现本机所有流量路由到 `clash` 代理、DNS 劫持等。
- 原理：[clash-verge-rev](https://www.clashverge.dev/guide/term.html#tun)、 [clash.wiki](https://clash.wiki/premium/tun-device.html)。

### `Mixin` 配置

```bash
$ clashmixin
😼 查看 mixin 配置

$ clashmixin -e
😼 编辑 mixin 配置

$ clashmixin -r
😼 查看 运行时 配置
```

- 运行时配置是订阅配置和 `Mixin` 配置的并集， `Mixin` 配置优先级大于订阅配置。
- 作用：防止更新订阅后丢失自定义配置。

### 卸载

```bash
sudo bash -c '. uninstall.sh; exec bash'
```

## 常见问题

### 下载失败、配置无效

- 下载失败：脚本使用 `wget`、`curl` 命令进行了多次[重试](https://github.com/nelvko/clash-for-linux-install/blob/035c85ac92166e95b7503b2a678a6b535fbd4449/script/common.sh#L32-L46)下载，如果还是失败可能是机场限制，请自行粘贴订阅内容到配置文件：[issue#1](https://github.com/nelvko/clash-for-linux-install/issues/1#issuecomment-2066334716)
- 订阅配置无效：[issue#14](https://github.com/nelvko/clash-for-linux-install/issues/14#issuecomment-2513303276)

### bash: clashon: command not found

- 原因：使用 `bash install.sh` 执行脚本不会对当前 `shell` 生效。
- 解决：当前 `shell` 执行下 `bash`。
- <details>

  <summary>几种运行方式的区别：</summary>

	- `bash` 命令运行：当前 `shell` 开启一个子 `shell` 执行脚本，对环境的修改不会作用到当前 `shell`，因此不具备 `clashon`
	  等命令。

	  ```bash
	  # 需要有可执行权限
	  $ ./install.sh
	  # 不需要可执行权限，需要读权限
	  $ bash ./install.sh
	  ```
	- `shell` 内建命令运行：脚本在当前 `shell` 环境中执行，变量和函数的定义对当前 `shell` 有效，`root` 用户推荐这种方式执行脚本。

	  ```bash
	  # 不需要可执行权限，需要读权限
	  $ . install.sh
	  $ source uninstall.sh
	  ```

  </details>

### 服务启动失败/未启动

- [端口占用](https://github.com/nelvko/clash-for-linux-install/issues/15#issuecomment-2507341281)
- [系统为 WSL 环境或不具备 systemd](https://github.com/nelvko/clash-for-linux-install/issues/11#issuecomment-2469817217)

## 引用

- [clash-linux-amd64-v3-2023.08.17.gz](https://downloads.clash.wiki/ClashPremium/)
- [Clash Web Dashboard](https://github.com/haishanh/yacd/releases/tag/v0.3.8)
- [Clash 全家桶下载](https://www.clash.la/releases/)
- [Clash 知识库](https://clash.wiki/)

## Todo

- [X] 定时更新配置
- [X] 😼
- [X] 适配其他发行版
- [X] 配置更新日志
- [X] Tun 模式
- [x] mixin 配置
- [ ] [bug / 需求](https://github.com/nelvko/clash-for-linux-install/issues)

## Thanks

[@鑫哥](https://github.com/TrackRay)

## 特别声明

1. 编写本项目主要目的为学习和研究 `Shell` 编程，不得将本项目中任何内容用于违反国家/地区/组织等的法律法规或相关规定的其他用途。
2. 本项目保留随时对免责声明进行补充或更改的权利，直接或间接使用本项目内容的个人或组织，视为接受本项目的特别声明。
