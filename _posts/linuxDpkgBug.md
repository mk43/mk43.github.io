---
title: 完美解决 Linux 的【dpkg： warning： files list file for package 'XXXXXXX' missing, assuming package has no files currently installed】Bug
date: 2017-11-04 11:27
tags:
	- linux
	- dpkg
---

## 0.前言
> 估计是之前动了或者损坏了 `/var/lib/dpkg/info` 里面的文件，每次执行 apt 类的命令总是输出一大段东西，在网上找了很多资料，有解决方案，但是不全。。。很多都是失败的。最后我发现 reinstall 可以解决，所以打算写个脚本执行文件。
> 

<!-- more -->

## 1.解决方法
### 1.1 创建一下三个文件
- fixit.py
- fix.sh
- txt

### 1.2 填写内容
先来最简单的 `fix.sh`，不用填写内容，是空文件。

接着就写 `txt`，直接把错误日志复制进去，如：

```
dpkg: warning: files list file for package 'libodbc1:amd64' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'dh-autoreconf' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'erlang-webtool' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libhtml-template-perl' missing; assuming package has no files currently installed
.......
dpkg: warning: files list file for package 'libvirt-dev:amd64' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'autopoint' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libconfig-general-perl' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'ubuntu-cloud-keyring' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'tgt' missing; assuming package has no files currently installed
dpkg: warning: files list file for package 'libfdt1:amd64' missing; assuming package has no files currently installed
```

下面写 `fixit.py`

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'Fitzeng'

import re

def main():
    fix = open('fix.sh', 'w+')
    for line in open("txt"):
        pkg = re.match(re.compile('''dpkg: warning: files list file for package '(.+)' '''), line)
        if pkg:
            cmd = "sudo apt-get install --reinstall " + pkg.group(1)
            fix.write(cmd + '\n')

if __name__ == "__main__":
    main()
```

### 1.3 执行命令
如果权限不够可以直接先 `chmod 777 *`，然后执行 `python fixit.py`，这时 `fix.sh` 就变成下面的样子了。


```
sudo apt-get install --reinstall libodbc1:amd64
sudo apt-get install --reinstall dh-autoreconf
sudo apt-get install --reinstall erlang-webtool
sudo apt-get install --reinstall libhtml-template-perl
sudo apt-get install --reinstall erlang-base

.......

sudo apt-get install --reinstall qemu-system-misc
sudo apt-get install --reinstall libvirt-dev:amd64
sudo apt-get install --reinstall autopoint
sudo apt-get install --reinstall libconfig-general-perl
sudo apt-get install --reinstall ubuntu-cloud-keyring
sudo apt-get install --reinstall tgt
sudo apt-get install --reinstall libfdt1:amd64
```

最后执行 `./fix.sh`。

然后就是等待执行结束了。

效果如下，一行一行 `dpkg: warning: ` 在减少。
![](debuging.png)
![](debuging1.png)


