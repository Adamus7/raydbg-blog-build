---
title: 解决 Homebrew 下载更新极慢的问题
pubDate: 2019-05-04 00:48:57
tags:
- Homebrew
---
近期使用 Homebrew 去下载安装软件的时候总是卡在 update 阶段，时间非常久，难以忍受。记录一下解决方法，
<!--more-->
# 症状
使用 Homebrew 安装软件的时候一直卡在 Update 阶段。同时发现从 github.com 下载文件也极度缓慢（几十 KB/s）。

# 问题定位
使用`brew update --verbose`观察 update 过程：
```
brew update --verbose
Checking if we need to fetch /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/caskroom/homebrew-fonts...
Fetching /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
remote: Enumerating objects: 337, done.
remote: Counting objects: 100% (337/337), done.
remote: Compressing objects: 100% (88/88), done.
remote: Total 298 (delta 221), reused 287 (delta 210), pack-reused 0
Receiving objects: 100% (298/298), 50.91 KiB | 39.00 KiB/s, done.
Resolving deltas: 100% (221/221), completed with 39 local objects.
From https://github.com/Homebrew/homebrew-core
   65a45a9..583b7f1  master     -> origin/master
remote: Enumerating objects: 179429, done.
remote: Counting objects: 100% (179429/179429), done.
remote: Compressing objects: 100% (56607/56607), done.
Receiving objects:   4% (7628/177189), 1.48 MiB | 8.00 KiB/s
```
发现 update 卡在从 github 仓库获取文件的过程。这个结果与手动从 github 下载文件慢的症状相互印证。

# 解决
由于问题主要是在国内网络环境 github 下载慢，因此尝试：
1. 更换使用国内的 homebrew 镜像源；
2. 使用代理访问 github.com。

## 更换 Homebrew 源
使用以下命令更换国内阿里云上的 homebrew 镜像：
```
# 替换 brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
# 替换 homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

# 替换 homebrew-bottles:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```
替换后，问题依旧，继续查看日志：
```
brew update --verbose
Checking if we need to fetch /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/caskroom/homebrew-fonts...
Fetching /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
From https://mirrors.aliyun.com/homebrew/homebrew-core
 + 583b7f1...8435590 master     -> origin/master  (forced update)
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Fetching /usr/local/Homebrew/Library/Taps/caskroom/homebrew-fonts...
remote: Enumerating objects: 179429, done.
remote: Counting objects: 100% (179429/179429), done.
remote: Compressing objects: 100% (56607/56607), done.
Receiving objects:   6% (11170/177189), 2.16 MiB | 30.00 KiB/s
```
可以看到由于`homebrew-cask`的仓库依然指向了 Github，这个过程还是慢。阿里云的 [镜像站](https://opsx.alibaba.com/mirror) 没有提供`homebrew-cask`，进一步搜索找到 [USTC 镜像站](http://mirrors.ustc.edu.cn/)，该站提供了`homebrew-cask`的源。使用上述同样的命令更换源：
```
# 替换 homebrew-cask.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```
测试发现问题解决。
> Note: Homebrew Bottles 源的更换方法详见 [这里](https://mirrors.ustc.edu.cn/help/homebrew-bottles.html)。
>
> 官方源地址：
>   https://github.com/Homebrew/brew.git
>   https://github.com/Homebrew/homebrew-core.git
>   https://github.com/Homebrew/homebrew-cask

## 使用代理
家里的路由器已经配置好了 SS 代理，只要简单把`github.com`加入到黑名单强制走 proxy 就可以了。然而代理速度一般，效果不及上述方法。同时为了保证在其他网络环境下的效率，保留方法一所做的修改。
