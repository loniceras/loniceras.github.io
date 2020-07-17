---
title: Haskell 开发环境搭建
date: 2020-02-10 12:43:59
tags: [编程]
categories: [编程]
mathjax: false
---
# 介绍
本文是在 Windows 10 操作系统下使用 [Stack](https://docs.haskellstack.org/en/stable/README/) 搭建在 VSCode 中的基于 [Haskell IDE Engine](https://github.com/haskell/haskell-ide-engine) ( HIE ) 的 Haskell 开发环境的介绍. Stack ( [GitHub页面](https://github.com/commercialhaskell/stack) ) 是用于开发 Haskell 的一个交互平台, 可以用于安装 Haskell 的编译器和各种库, 也可以用于构建和测试项目. Stack 对库的版本管理一定程度上依赖于[解析器](https://docs.haskellstack.org/en/stable/yaml_configuration/#resolver)的选择, 解析器支持 [LTS 快照](https://github.com/commercialhaskell/lts-haskell), [Stackage Nightly快照](https://github.com/commercialhaskell/stackage-nightly)等多种格式对版本进行指定, 一个快照下的一个库只能拥有一个版本号, 可在 [Stackage 网站](https://www.stackage.org/)上查询不同 GHC 对应的最新 LTS 快照. 更多有关 Stack 的教程可以参考[用户文档](https://github.com/commercialhaskell/stack/blob/master/doc/GUIDE.md). HIE 是一个针对不断增长的 Haskell 工具的一个通用接口, 它为编辑器和 IDE 提供了功能齐全的语言服务器协议服务, HIE 目前支持的最低 Stack 版本为 2.1.1, 它仍然在不断开发中.
<!--more-->
# 配置 Stack
从 [Stack 官网](https://docs.haskellstack.org/en/stable/install_and_upgrade/#windows) 下载 Stack 的安装包并进行安装. 安装完毕后, 以管理员身份运行 CMD , 执行 `stack setup` , 直至生成 `C:\sr\config.yaml`. 随后打断 CMD 的操作, 在 `config.yaml` 文件中进行换源, 可以使用的 Hackage 镜像有:
https://mirrors.tuna.tsinghua.edu.cn/help/hackage/
```
package-indices:
- download-prefix: http://mirrors.tuna.tsinghua.edu.cn/hackage/
  hackage-security:
    keyids:
    - 0a5c7ea47cd1b15f01f5f51a33adda7e655bc0f0b0615baa8e271f4c3351e21d
    - 1ea9ba32c526d1cc91ab5e5bd364ec5e9e8cb67179a471872f6e26f0ae773d42
    - 280b10153a522681163658cb49f632cde3f38d768b736ddbc901d99a1a772833
    - 2a96b1889dc221c17296fcc2bb34b908ca9734376f0f361660200935916ef201
    - 2c6c3627bd6c982990239487f1abd02e08a02e6cf16edb105a8012d444d870c3
    - 51f0161b906011b52c6613376b1ae937670da69322113a246a09f807c62f6921
    - 772e9f4c7db33d251d5c6e357199c819e569d130857dc225549b40845ff0890d
    - aa315286e6ad281ad61182235533c41e806e5a787e0b6d1e7eef3f09d137d2e9
    - fe331502606802feac15e514d9b9ea83fee8b6ffef71335479a2e68d84adc6b0
    key-threshold: 3 # number of keys required

    # ignore expiration date, see https://github.com/commercialhaskell/stack/pull/4614
    ignore-expiry: no
```
https://docs.haskellstack.org/en/stable/yaml_configuration/#package-indices
```
package-indices:
- download-prefix: https://hackage.haskell.org/
  hackage-security:
    keyids:
    - 0a5c7ea47cd1b15f01f5f51a33adda7e655bc0f0b0615baa8e271f4c3351e21d
    - 1ea9ba32c526d1cc91ab5e5bd364ec5e9e8cb67179a471872f6e26f0ae773d42
    - 280b10153a522681163658cb49f632cde3f38d768b736ddbc901d99a1a772833
    - 2a96b1889dc221c17296fcc2bb34b908ca9734376f0f361660200935916ef201
    - 2c6c3627bd6c982990239487f1abd02e08a02e6cf16edb105a8012d444d870c3
    - 51f0161b906011b52c6613376b1ae937670da69322113a246a09f807c62f6921
    - 772e9f4c7db33d251d5c6e357199c819e569d130857dc225549b40845ff0890d
    - aa315286e6ad281ad61182235533c41e806e5a787e0b6d1e7eef3f09d137d2e9
    - fe331502606802feac15e514d9b9ea83fee8b6ffef71335479a2e68d84adc6b0
    key-threshold: 3 # number of keys required

    # ignore expiration date, see https://github.com/commercialhaskell/stack/pull/4614
    ignore-expiry: true
```
换源完成后, 继续执行 `stack setup`. 初始化完成后使用 `stack update` 更新 `index.tar.gz` 文件. 更新该文件时可能会发生哈希检验错误, 参考 [#3052](https://github.com/commercialhaskell/stack/issues/3052) 和 [#3771](https://github.com/commercialhaskell/stack/issues/3771), 反复进行删除 `C:\sr\pantry\hackage` 下的文件并再次执行 `stack update` 的操作直至成功为止. 如果该错误始终无法解决, 这可能是由于镜像源本身的错误造成的, 可尝试使用别的镜像源. 在使用第二个镜像源时, 使用代理可加快下载速度. 以 SSR 代理的默认端口为例, 在 CMD 中执行 `set http_proxy=http://127.0.0.1:1080` 和 `set https_proxy=http://127.0.0.1:1080`. 注意只能在 CMD 中设置, 此法对 Powershell 无效.

# 配置 HIE
新建一个短路径 ( 如 `C:\hie` ) 用于克隆 HIE 仓库, 若要选择长路径可参考[此处](https://github.com/haskell/haskell-ide-engine#windows-specific-pre-requirements). 克隆是通过递归子模块的方式进行的: `git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules`. 克隆完成后, 可通过 `stack ./install.hs help` 查看操作指南, 此过程需要等待一段时间以安装一些必要文件. 等出现操作指南后, 可使用 `stack ./install.hs hie-8.6.5` 进行安装 HIE. 笔者没有采用 `stack ./install.hs hie` 进行安装, 若选择此项则安装的是支持当前最新的 GHC 版本 GHC 8.8.2 的 HIE. 在安装 HIE 过程中安装 `network` 库时可能出现拒绝许可的错误, 参考 [#5014](https://github.com/commercialhaskell/stack/issues/5014), 可尝试将 CMD 的字符编码改为 utf-8 进行解决, 即在 CMD 中输入 `chcp 65001` 后继续执行 `stack ./install.hs hie-8.6.5`. 安装完成后, 在 VSCode 中安装 `Haskell Language Server` 插件. HIE 支持在悬停状态下从 haddock 中获取文档, 这一功能需要使用到 `hoogle` 数据库, 根据[文档](https://github.com/haskell/haskell-ide-engine#docs-on-hovercompletion)说明, 可以在 `C:\hie` 下使用 `stack --stack-yaml=stack-8.6.5.yaml exec hoogle generate` 来生成 `hoogle` 数据库. 现在使用 VSCode 打开一个使用 Stack 构建的项目文件夹时, 便能体验基于 HIE 的开发环境. 打开一个 `hs` 文件, 在 HIE 初始化的过程中, 可能会产生 [#1616](https://github.com/haskell/haskell-ide-engine/issues/1616) 错误, 可以通过先 `stack build` 整个项目文件, 再在 VSCode 中打开 `hs` 文件来解决.

# 后记
笔者根据该[建议](https://github.com/bitemyapp/learnhaskell/blob/master/guide-zh_CN.md)选择使用 Stack 代替 Haskell Platform 进行 GHC 和库的管理, 便有了这次重新搭建开发环境的尝试. 中文社区基本没有可用的博客和说明, 大都语焉不详或早已过时. 尽管笔者仔细参考了官方文档, 在这次配置过程中仍踩了很多很多很多很多很多很多的坑, 但经过不断查阅社区资料后最终还是配置成功了. 笔者从多而繁杂的解决过程中选择了一些简单明了的方案总结在此文中, 希望能为同好们节省尝试的时间.