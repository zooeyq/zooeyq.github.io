---
title: Ubuntu虚拟机踩坑记录
date: 2026-10-18 22:57:46
categories:
  - 技术笔记
tags:
  - Ubuntu
  - VirtualBox
    - WebGL
    - Node.js
    - NVM
---
最近一直在折腾 VirtualBox 里的 Ubuntu 系统。今天使用过程中遇到了两个问题，一个是 Chrome 打开 DeepSeek 网页时报错，另一个是昨天安装好的 Node.js 今天突然无法使用。

由于自己不太清楚原因，所以我把报错信息发给了 ChatGPT，让它帮我一步一步排查。这里记录一下整个过程。

<!-- more -->

## 一、Chrome 打开 DeepSeek 出现 Application error

### 1. 遇到问题

今天我在 VirtualBox 的 Ubuntu 系统中使用 Chrome 打开 DeepSeek 网页，结果出现了下面的提示：

```text
Application error: a client-side exception has occurred
(see the browser console for more information).
```

因为之前配置过 Clash 代理，所以我一开始也不确定是不是网络的问题。

于是我把错误信息发给了 ChatGPT。

ChatGPT 告诉我，这段提示的意思是网页前端的 JavaScript 在运行时出现了异常，并不一定是网络连接失败。要想知道具体原因，需要查看浏览器控制台。

### 2. 查看浏览器控制台

按照 ChatGPT 的提示，我在 Chrome 中按下 `F12`，打开开发者工具，进入 Console，然后刷新网页。

控制台出现了以下错误：

```text
THREE.WebGLRenderer: A WebGL context could not be created.

THREE.WebGLRenderer: Error creating WebGL context.

Error: Error creating WebGL context.
```

我把这些错误信息复制给 ChatGPT。

它分析后告诉我，问题很可能与 WebGL 有关。

WebGL 是浏览器用于图形渲染的一项技术，而 Three.js 是一个 JavaScript 3D 图形库。

网页在初始化 Three.js 的渲染器时，无法创建 WebGL 上下文，因此抛出了异常。

日志中还出现了：

```text
GL_RENDERER = ANGLE (Mesa, llvmpipe ...)
```

ChatGPT 解释说，`llvmpipe` 是 Mesa 的软件渲染器，说明当前图形环境使用了软件渲染路径。

不过，软件渲染不代表一定无法使用 WebGL，所以还需要继续检查。

### 3. 检查 WebGL 状态

接下来，ChatGPT 让我在 Chrome 地址栏输入：

```text
chrome://gpu
```

打开后找到 Graphics Feature Status，检查 WebGL 的状态。

结果发现：

```text
WebGL: Disabled
```

我把结果告诉 ChatGPT，它认为 WebGL 不可用与网页报错直接相关，并建议继续检查 VirtualBox 的显示配置。

### 4. 检查 VirtualBox 配置

按照 ChatGPT 的提示，我检查了 VirtualBox 的显示设置，发现：

| 配置项 | 当前状态 |
| --- | --- |
| 图形控制器 | VMSVGA |
| 3D 加速 | 未开启 |
| Chrome WebGL | Disabled |

ChatGPT 告诉我，VMSVGA 是适合 Ubuntu 的图形控制器，但没有开启 3D 加速可能会影响 WebGL 的使用。

它建议我先关闭 Ubuntu，然后进入：

**VirtualBox → 设置 → 显示 → 屏幕**

尝试调整为：

```text
图形控制器：VMSVGA
显存大小：128 MB
启用 3D 加速：开启
```

重新启动 Ubuntu 后，再进入：

```text
chrome://settings/system
```

检查 Chrome 是否开启图形加速。

然后通过：

```text
chrome://gpu
```

重新查看 WebGL 状态。

如果仍然无法使用 WebGL，还可以进一步检查 VirtualBox Guest Additions 和 Ubuntu 的图形驱动。

目前我已经通过控制台确认了 WebGL 上下文创建失败的问题，但还没有验证开启 3D 加速后能否彻底解决。

## 二、明明安装了 Node.js，却提示找不到命令

### 1. 遇到问题

昨天我在 Ubuntu 中通过 NVM 安装了 Node.js，当时已经可以正常使用。

但今天重新打开终端，执行：

```bash
node -v
```

系统却提示：

```text
找不到命令 “node”
```

继续执行：

```bash
npm -v
```

结果也一样。

我觉得很奇怪，明明昨天已经安装成功，为什么今天就找不到了？

于是我再次询问 ChatGPT。

### 2. 检查 NVM

ChatGPT 告诉我，Node.js 不一定被卸载了，也可能是 NVM 没有正确激活 Node.js 版本。

它让我先执行：

```bash
nvm --version
```

结果显示：

```text
0.40.7
```

说明 NVM 本身可以正常使用。

接着执行：

```bash
ls -la ~/.nvm
```

发现 NVM 的安装目录仍然存在。

然后执行：

```bash
whoami
```

输出：

```text
vae
```

说明当前仍然是原来的 Ubuntu 用户。

我把这些结果发给 ChatGPT，它判断 NVM 已经正常安装和加载，下一步需要确认 Node.js 的版本是否还在。

### 3. 找到真正的原因

ChatGPT 让我执行：

```bash
nvm ls
```

输出中包含：

```text
v22.16.0
v22.23.3

default -> lts/* (-> N/A)
```

看到这个结果后，ChatGPT 告诉我，Node.js 实际上没有丢失，本地仍然安装着两个版本。

问题出在这一行：

```text
default -> lts/* (-> N/A)
```

`lts/*` 表示默认使用最新的长期支持版本，而 `N/A` 表示这个别名当前对应的版本在本地不可用。

因此，虽然 Node.js 已经安装，但终端没有自动激活可用的版本。

这才导致 `node` 和 `npm` 命令无法使用。

### 4. 解决方法

ChatGPT 建议我直接使用本地已经安装的 Node.js 版本，不需要重新安装。

首先执行：

```bash
nvm use 22.23.3
```

然后检查：

```bash
node -v
npm -v
```

为了以后打开终端时能够自动使用这个版本，还需要设置默认版本：

```bash
nvm alias default 22.23.3
```

接着执行：

```bash
nvm use default
```

ChatGPT 还向我解释了这两条命令的区别：

`nvm alias default` 是指定默认使用哪个 Node.js 版本。

`nvm use default` 是在当前终端激活默认版本。

最后重新打开终端，执行：

```bash
node -v
npm -v
nvm current
```

如果能够正常显示版本号，说明默认版本已经成功加载。

目前已经确认本地安装的 Node.js 版本仍然存在，默认版本别名也确实指向了一个未安装的版本。至于修改配置后能否在新终端中自动加载，还需要实际验证。

### 5. 一点收获

这次我原本以为 Node.js 消失了，甚至考虑重新安装。但通过 ChatGPT 的引导，逐步检查 NVM、安装目录和默认版本后，才发现只是版本配置的问题。

以后再遇到类似情况，可以先执行：

```bash
nvm ls
```

查看已安装的 Node.js 版本，而不是直接重新安装。

这两个问题也让我体会到，遇到报错时，先把错误信息看清楚，再借助工具逐步排查，比盲目修改配置更有效。
