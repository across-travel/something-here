---
title: 使用nvm管理不同版本的node与npm
date: 2016-06-10 21:40
tags:
 - Node
---

在我们的日常开发中经常会遇到这种情况：手上有好几个项目，每个项目的需求不同，进而不同项目必须依赖不同版的 NodeJS 运行环境。如果没有一个合适的工具，这个问题将非常棘手。

## nvm 与 n 的区别

node 版本管理工具还有一个是 n 命令，n 命令是作为一个 node 的模块而存在，而 nvm 是一个独立于 node/npm 的外部 bash 脚本，因此 n 命令相比 nvm 更加局限。

由于 npm 安装的模块路径均为 /usr/local/lib/node_modules，当使用 n 切换不同的 node 版本时，实际上会共用全局的 node/npm 目录。 因此不能很好的满足『按不同 node 版本使用不同全局 node 模块』的需求。

## 卸载全局安装的 node/npm

在官网下载的 node 安装包，运行后会自动安装在全局目录，使用过程中经常会遇到一些权限问题，所以推荐卸载全局安装的 node/npm。

## Linux 安装

我们并不一定要先卸载原有的 NodeJS。当然我们推荐还是先卸载掉比较好。另外，你还需要 C++ 编译器，Linux 发行版一般不用担心，像 Ubuntu 都可以直接用 build-essential 套件

在 Linux 中：（如果是 Debian 发行版）

```bash
sudo apt-get install build-essential
```
然后我们可以使用

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

或者

```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

从远程下载 install.sh 脚本并执行。注意这个版本年数字 v0.33.0 会随着项目开发而变化。随时通过[官方最新安装命令](https://github.com/creationix/nvm#install-script)来检查最新安装版本是有好处的。

## 安装多版本 node/npm

例如，我们要安装4.2.2版本，可以用如下命令：

```bash
nvm install 4.2.2
```
nvm 遵守语义化版本命名规则。例如，你想安装最新的 4.2 系列的最新的一个版本的话，可以运行：
```bash
nvm install 4.2
```
nvm 会寻找 4.2.x 中最高的版本来安装。

你可以通过以下命令来列出远程服务器上所有的可用版本：
```bash
nvm ls-remote
```

## 在不同版本间切换

每当我们安装了一个新版本 Node 后，全局环境会自动把这个新版本设置为默认。

nvm 提供了 nvm use 命令。这个命令的使用方法和 install 命令类似。

例如，切换到 4.2.2：
```bash
nvm use 4.2.2
```
切换到最新的 `4.2.x``：

```bash
nvm use 4.2
```

切换到最新版：
```bash
nvm use node
```

每次执行切换的时候，系统都会把 node 的可执行文件链接放到特定版本的文件上。

我们还可以用 nvm 给不同的版本号设置别名：
```bash
nvm alias awesome-version 4.2.2
```

我们给 4.2.2 这个版本号起了一个名字叫做 awesome-version，然后我们可以运行：

```bash
nvm use awesome-version
```

下面这个命令可以取消别名：

```bash
nvm unalias awesome-version
```
另外，你还可以设置 default 这个特殊别名：

```bash
nvm alias default node
```

## 列出已安装实例

```bash
nvm ls
```

上面绿色箭头是当前正在使用的版本，下面列出的还有设置过的别名。

## 在项目中使用不同版本的 Node

我们可以通过创建项目目录中的 .nvmrc 文件来指定要使用的 Node 版本。之后在项目目录中执行 nvm use 即可。.nvmrc 文件内容只需要遵守上文提到的语义化版本规则即可。另外还有个工具叫做 avn，可以自动化这个过程。

## 在多环境中，使用npm

每个版本的 Node 都会自带一个不同版本的 npm，可以用 npm -v 来查看 npm 的版本。全局安装的 npm 包并不会在不同的 Node 环境中共享，因为这会引起兼容问题。它们被放在了不同版本的目录下，例如 ~/.nvm/versions/node/<version>/lib/node_modules</version> 这样的目录。这刚好也省去我们在 Linux 中使用 sudo 的功夫了。因为这是用户的主文件夹，并不会引起权限问题。

但问题来了，我们安装过的 npm 包，都要重新再装一次？幸运的是，我们有个办法来解决我们的问题，运行下面这个命令，可以从特定版本导入到我们将要安装的新版本 Node：

```bash
nvm install v5.0.0 --reinstall-packages-from=4.2
```

## 其他命令

直接运行特定版本的 Node

```bash
nvm run 4.2.2 --version
```

在当前终端的子进程中运行特定版本的 Node

```bash
nvm exec 4.2.2 node --version
```
确认某个版本Node的路径

```bash
nvm which 4.2.2
```