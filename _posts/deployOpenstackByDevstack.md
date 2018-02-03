---
title: 一个小白使用 devstack 部署 openstack 的心路历程
date: 2017-11-04 11:27
tags:
	- devstack
	- openstack
---

## 0.前言
> 作为一个想要入门云计算的人来说，大多数人的第一步就是学习 openstack，而学习 openstack 的人面临的第一步就是第一个‘难题’，使用自动化部署工具 devstack 部署 openstack 环境。第一次接触这个东西，花了两天多时间在 Ubuntu Server 和 Ubuntu Desktop 的 16.04 版本上成功部署。有很多人说 Desktop 版有很多坑，亲测问题确实比 Server 版多，但都是可解决的。其中最对初学者不友好的就是网络问题，下面会介绍为什么。所以如果你自己买了国外的 VPS 那就好办了，因为有个骚操作要用到，如果你网速好可能你遇不到这些问题就可以忽略。
> 
> 下面会分两个部分介绍，都会将 Server 和 Desktop 上的部署过程描述一遍。
> 教程推荐 [官方的 Doc](https://docs.openstack.org/devstack/latest/)和[避坑指南](https://zhuanlan.zhihu.com/p/28996062)
> 教程这个东西对于初学者不宜太多，容易乱，只要有一个正确的执行框架就好。碰到其他 bug 直接 Google 就好。

<!-- more -->

然后介绍下我的环境吧

- Mac 10.12.6
- VirtualBox 5.1.28
- Ubuntu Server 16.04 4G+20G (临时测试 devstack，听说坑少)
- Ubuntu Desktop 16.04 4G+80G (平时使用)
- VPS(最好有) (由于是乞丐版，不适合直接部署和平时学习)

## 1.Ubuntu Server 版
### 安装 Ubuntu Server
首先肯定是要在 Virtual Box 安装 Ubuntu Server 了，这一步略过。相信你已经是接触过一段时间虚拟机的人了，但是一点注意，竟可能分多一点内存和硬盘。由于我不打算日后再这 Server 版使用，所以我的配置是 4G + 20G

### SSH 登录虚拟机
当你创建完成之后面临的一个问题就是那个界面太丑了。。。所以如果可以在宿主机上操作就好了，SSH 正好满足你。
至于 SSH 不通使用不了的自己查查资料吧，这里我主要介绍网卡配置，我使用了两个网卡：
第一个：
![](wk1.png)
![](wk1port.png)
做端口映射，将主机的 2222 映射到虚拟机的 22，这条是为了以后使用 SSH。
第二个：
![](wk2.png)
配完之后再在全局配置中设置你所选中的网卡启用 DHCP。对于网卡各种连接模式不熟的可以查查资料了解一下。

然后连接就是直接在主机下使用  

```
ssh -p 2222 fitzeng@127.0.0.1
```

fitzeng 改成你的用户名。
如果你出现各种问题连不上可以注意一下两点：
1.防火墙
2.把 `~/.ssh` 文件夹下的 `known_hosts` 文件删了再重连

### 开始部署
> 这里的主教程以官方提供的为准，并且那些注意点我会更新。

部署的脚本要求是拥有 root 权限的非 root 用户。

```
sudo useradd -s /bin/bash -d /opt/stack -m stack

echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack

cd /opt/stack
git clone https://git.openstack.org/openstack-dev/devstack
cd devstack
```

如果上面 clone 太慢或者 clone 不下来的话可以试试 github 的源。

```
git clone https://github.com/openstack-dev/devstack.git
```

然后就是添加配置了，如果不懂推荐直接使用官方页面介绍的。或者使用以下命令

```
cp samples/local.conf ./
vim local.conf
```

如果你幸运，讲道理最后执行 `./stack.sh` 直接一路到底。。。但是还有很多坑正在等待着我们。
但但是有一个很好的是他的 Log 和报错十分清新，很快可以定位问题所在，有时候直接搜 Log 都会出现解决方法。
如果脚本直接退出提示没有 HOST_IP。那么直接在 `local.conf` 后面添加

```
HOST_IP=x.x.x.x
GIT_BASE=https://github.com
```

HOST_IP具体是什么在你的虚拟机上 ifconfig 查看。然后推荐把 git 源换成 github 的。
这里你可以检测一下你的源有没有问题 `apt-get update` 有的话直接把有问题的源在 `/etc/apt/sources.list.d/` 目录下移除，移除前建议备份一下。然后推荐 `apt-get upgrade` 一下，Python 版本保持默认的 2.7.X 就好，如果出现什么和 Python 3.4 不匹配的 Log 直接忽略。如果你换成 3.4 很多库会出问题。如果你是 Python 3.X，可以把 `/user/bin/` 下的 Python2.X 链接到该目录下的 Python 文件。这时执行 `python -V` 就能看到结果了。

但但但是，上面只是解决了有形的 Bug，还有就是无形的 Bug，你将面临网络问题，如果你想顺畅点可以直接更换源。
修改 pip 源

```
mkdir ~/.pip
vim ~/.pip/pip.conf

填入：
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple
```

修改 sources.list：

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list

填入：
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

都改成 aliyun 的。到这时候如果你的网络没什么问题，可能会出点环境小问题，dkpg 和各种包等之类的问题，一搜网上基本都有解决方案。

但但但是，如果你的网速下载某些包不超 10Kb/s 那就要用骚操作了。。。因为会一直卡着，网一断又得重新开始，先 unstack，clean 再 stack，十分不友好。出现问题大多是在下载某个 git 仓库和某些包的时候。尤其是 `nova` `horizon` 之类的，大小到了 300+M。


这里介绍一个方法：
思路是先 SSH 上你的国外 VPS，下载你的 git 仓库或其他文件。然后再 SCP 到你的虚拟机上。主要是这样不会中断，而且无形中就可以是多线程操作，开几个终端 SCP 好几个文件。
看看速度对比效果吧，
虚拟机上下载：
![](horizon2.png)
VPS 上下载：
![](horizon.png)
之后自己 SCP 就好了

```
sudo scp -P 10800 -r root@xx.xx.xx.xx:/fitzeng/horizon /etc/stack/
```
-r 是 cp 文件夹，然后端口，IP 填你自己的后面跟目录。这里可能也有点慢，但是比之前的好而且稳定。

![](nova.png)
![](cirros.png)
这一切操作都源于友好的 log 机制，看上面的图片我们可以知道下载地址和存放目录，所以，知道这些手段就多了起来。
网速够快也可以直接在本地 clone。
![](nova2.png)

有了这些操作基本就意味着你解决了网络问题，借助 google 基本可以解决其他库和环境的问题。
成功图上传一波：
![](finish.png)


## 2.Ubuntu Desktop 版
基本步骤是和前面一致的，出的问题可能就是你之前在 Ubuntu 上装过各种软件(我装的 Sogou 输入法，里面的 fcitx 源影响了速度，甚至有时候直接卡这不动)，更改了软件源或者做过其它的工具更改，按照前面的配置亲测可行。如果你之前在 Ubuntu Server 版上装过，直接把文件 SCP 过来，如果虚拟机之间不能通讯，可以先 SCP 到宿主机，再从宿主机通过文件共享的方式共享到 Ubuntu Desktop。
然后运行就可以了，有了前面的基础就很简单了。

那就看直接看结果吧：
![](d1.png)
![](d2.png)
![](d3.png)
![](d4.png)

## 3.后记
说实话，这不太算技术文章，纯属个人记录。本来不太想写，但是感觉国内环境对开发者有点不友好，如果这篇文章能对初学者有部分帮助我就满意了，能够使初学者继续学习下去。然后这是部署完之后写的，部署的过程远不如写的这么轻松，但是我现在有信心去解决部署过程中碰到的问题，这才是重点。希望你也是。每个人的环境都不一样，出现的问题也不可能一样，所以如果你照上面做了还没有解决可以留言大家一起讨论。

最后：
多谢阅读
祝大家一遍过 ^_^