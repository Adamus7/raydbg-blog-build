---
title: 一种绕过“反抓包”策略的 HTTP 报文捕获方法
pubDate: 2020-04-14 18:26:23
tags:
- HTTP
- Charles
- Debugging
---

一般情况下的抓包模式是一种称之为“中间人”抓包的形式，既通过系统 Proxy 配置把网络请求转发到一个充当中间人的程序上（如 Charles、FIddler），再由该抓包程序进行网络请求的转发。这个过程中，抓包软件可以对 HTTP 层面的数据进行记录和展示。
“反抓包”是比较常见的 App 网络安全策略。对于上述的这种抓包模式，一种常见的反抓包策略是强制 App 不使用系统配置的 Proxy，由此避免网络流量被抓包软件截获。
这类策略，给日常的问题排查或研究带来了一些不便。本文将介绍一种绕过该安全策略的通用方法，实现对目标 App 的网络包捕获。
<!--more-->
![1.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/1.png)


# 1. 基本原理
第 1 节简单介绍了一般抓包原理和“反抓包”策略。可以看到，绕过“反抓包”策略的关键是如何转发流量到“中间人”设备上。
现有的 App 市场上有一类可以帮助用户实现连接私有特殊协议的 VPN 工具软件，如 Kitsunebi。这类 App 的基本原理是：1. 构建一个 OS 级别的 VPN 通道；2. 构建一个本地的代理服务客户端程序；3. 通过 VPN 通道截获本地流量，转发到本地代理服务的客户端程序上，再由该客户端通过指定的代理协议转发流量到远程的代理服务器上，打通通信隧道。
我们注意到，VPN 通道可以实现全局的 TCP 层面数据转发，加上远程代理服务器是由客户端软件配置的，因此，可以实现不依赖 OS Proxy 配置的流量转发。另外，这类 VPN 软件一般支持 SOCKS5 代理协议，而“中间人”抓包软件往往也支持 SOCKS5 代理服务。结合这两个特性，可以实现的抓包拓扑为：目标 App 的通信通过 VPN 通道转发到本地代理客户端，由本地代理客户端转发流量到代理服务器上，该代理服务器实际是我们的抓包软件，再由抓包软件把请求转发到目标应用服务器上。基本原理如下图所示：
![2.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/2.png)
在这个流程中，抓包软件可以截获流量，从而实现对 HTTP 的记录和展示，以及 HTTPS 报文的解密。

# 2. 实施步骤示例
> 物料准备：
> - 手机（以 iOS 为例）
> - 一个支持 SOCKS5 代理能力的工具 App（以 Kitsunebi 为例）
> - “中间人”设备和软件（以 Mac 和 Charles 为例）

## 2.1 抓包工具配置
以 Charles 为例，主要的配置步骤就是打开 SOCKS 代理服务：
![3.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/3.png)

## 2.2 客户端配置
以 iOS/Kitsunebi 为例，基本步骤如下：
1. 在 Kitsunebi 中选择全局转发模式（简单高效）：
![4.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/4.png)

2. 新建一个 SOCKS 代理服务配置，并指向 Charles 机器 IP 和已配置的 SOCKS 服务端口：
![5.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/5.png)

3. 配置启用该 SOCKS 代理：
![6.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/6.png)

4. 启用 Kitsunebi VPN 服务：
![7.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/7.png)

## 2.3 验证
此时，原来配置了“反抓包”策略的 HTTP 流量应该可以被 Charles 捕获：
![8.png](/images/2020-04-14-How-To-Capture-HTTP-Traffic-When-the-App-Is-Not-Applying-the-OS-Proxy-Configuration/8.png)

# 3 小结
这里介绍了一种通用的绕过“强制不走 OS Proxy 反抓包策略”的 HTTP 报文捕获方法，解释了原理和一种实施途径。在实践中，可以结合实际情况通过不同的工具组合在不同的平台上实现该方法。