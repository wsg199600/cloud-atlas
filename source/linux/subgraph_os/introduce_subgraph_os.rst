.. _introduce_subgraph_os:

=======================
Subgraph OS简介
=======================

我们知道微软有一个RDP远程桌面解决方案，提供了Windows客户端访问Server上运行的桌面方式。这种集中化管理可以让系统管理员将Server上的应用加固，所有用户的数据都存放在中央服务器上，所有计算都在中央服务器上运行，对客户端硬件要求极低。

虽然RDP协议开放，基于xrdp有跨平台的解决方案，Linux也可以使用 :ref:`xrdp` 实现中心化的数据存储和计算，但是RDP的桌面很难融入本地运行操作系统。看上去总是本地一个窗口中嵌套的桌面，使用体验不佳。

:ref:`xpra` 是一种融合了VNC远程不间断访问和X Window运行计算在服务器显示在本地的混合技术，开源并且活跃开发。Xpra能够实现显示、音频、打印等远程工作，并且具有X
window显示窗口完全融合到本地操作系统的优点。

我在接触到Xpra之后，首先想到的就是在服务器上运行大型Linux开发工具，能够随时随地通过轻量级的Xpra客户端连接到服务器上进行移动开发。我甚至想象今后的自由开发者也可以采用租用云计算的虚拟机，只需要一台简单的低配置笔记本，坐在星巴克里，联上网络就可以随时继续之前暂停的开发工作，完全不需要重新建立开发环境。

.. note::

   要实现一个完整的具备企业级认证，按需配置的远程工作环境，Xpra只是基础之一，需要综合多项技术才能实现企业级的云计算服务。我觉得云计算SaaS有这方面市场。

什么是Subgraph OS
===================

.. note::

   Subgraph OS的目标是安全，从 `Exploring Subgraph OS <https://subgraph.com/sgos/graph/index.en.html>`_ 初步来看，这个操作系统的架构：

   - 本地操作系统中把所有应用程序都通过容器进行
   - 应用程序通过Xpra映射到本地操作系统图形界面中，所有计算依然在容器中，防止应用逃逸到主操作系统
   - 采用公用的Tor代理(MetaProxy)实现应用程序的强制匿名安全网络访问
   - 本地操作系统文件系统加密并且加强内核安全

   从方案介绍来看，和我构想的中心化应用运行不同：我构想的是基于云计算的应用程序，是从企业和系统管理员角度增强安全和高可用；Subgraph OS则将跨我所构想的跨网络分布式运行架构压缩到本地操作系统中，专注于安全隔离性。

   不过，殊途同归，可以借鉴Subgraph OS的技术堆栈体系。

.. figure:: ../../_static/linux/subgraph_os/subgraph_os.png
   :scale: 75

应用容器层(本地)
-----------------

Subgraph OS在用户本地操作系统采用Linux Namespace结合 :ref:`xpra` 下线应用程序和主操作系统的隔离。应用程序运行在容器中，这样不仅防止访问未授权本地文件系统，也阻止了从其他X11应用程序嗅探键盘记录以及不必要的网络访问。本地操作系统采用了加固的内核来确保应用更难从容器中逃逸：

- Subgraph OS使用了Tor Browser(提供匿名保护的特殊配置Firefox浏览器)
- Subgraph OS的电子邮件客户端是Icedove(默认配置了Enigmail和TorBirdy扩展，运行在沙箱采用加固安全PaX保护防御恶意攻击)
- Subgraph OS将PDF阅读器隔离在没有网络的应用容器中，限制潜在风险的PDF访问主机文件或发起提供攻击的网络连接

.. note::

   Subgraph OS的安全方案基于:

   - 应用程序主体在集中化的远程服务器上运行，可以让系统管理员能够极大加强安全防范，实现文件安全扫描，备份和故障恢复
   - 本地仅提供应用程序通过Xpra显示，并且本地应用采用容器隔离，进一步加强安全

元代理(MetaProxy)
--------------------

由于不是所有应用程序都能够通过Tor实现安全通讯，并且一些应用程序需要非常特殊配置。为了方便用户使用，Subgraph的 ``MetaProxy`` 提供透明的Tor转发，通过Tor代理避免了复杂的应用程序配置，应用程序和外部通讯是通过MetaProxy转发，确保每个应用程序通过Tor网络的不同线路通讯。

防火墙(Firewall)
-------------------

Subgraph OS应用程序防火钱是用来控制哪些应用程序能够发起对外连接。当一个未知的应用程序试图对外连接，系统就会提示用户是否允许应用临时或永久对外发起连接。这种控制可以防范恶意程序。

.. note::

   请注意Subgraph OS的防火墙不是常规意义的防范外部进入连接的防火墙，而是控制未经授权的对外访问，这对很多木马以及恶意程序有一定防范作用。

增强内核(Hardened Kernel)
---------------------------

Subgraph OS默认使用了一种Grsecurity-enabled内核，这种增强安全采用了PaX的安全漏洞防范技术，避免应用程序和操作系统内核缓存移除等漏洞。

加密文件系统(Encrypted Filesystem)
-----------------------------------

Subgraph OS安装时会要求用户使用Linux的trusted dm-crypt/LUKS机制进行全盘加密。另外，系统内存在关机时会擦除以便防范冷启动攻击窃取加密密钥。

Tor匿名加密网络(Tor Network)
-----------------------------

默认情况下，Subgraph OS只通过Tor网络通讯。使用Tor浏览器进行匿名web浏览，同时其他强制通过Tor通讯的应用程序提高安全。这意味着所有外出连接都是匿名的。如果用户需要非匿名网络浏览，也提供了和其他操作系统相同的普通非匿名浏览器。

参考
======

- `Exploring Subgraph OS <https://subgraph.com/sgos/graph/index.en.html>`_
