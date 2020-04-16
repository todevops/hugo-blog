---
title: 使用 oh-my-posh 美化 PowerShell
date: 2019-01-05
categories:
  - tools
tags:
  - windows
  - powershell
---

使用 oh-my-posh 美化 PowerShell
<!--more-->

英文字体是支持PowerLine的DejaVuSansMono字体，中文字体是文泉驿等宽微米黑字体），并将终端字体设置为支持PowerLine的字体。
然后开始安装oh-my-posh（该步骤可能需要某种“较为稳定”的网络环境）。在管理员权限的PowerShell下执行指令
```powershell
Set-ExecutionPolicy Bypass
```
该指令旨在允许加载并运行任意脚本。可能会造成安全问题，但是只要有杀毒软件在就无需担心，毕竟没有人会无聊到对一个普通的计算机用户进行针对性攻击。

然后安装oh-my-posh的依赖和oh-my-posh本身

```powershell
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

安装完成之后，可以通过
```powershell
Import-Module oh-my-posh
```

来尝试启用这个模组。之后就可以使用

```powershell
Set-Theme主题名
```

这种格式的指令来切换不同的显示风格。示例图中的主题是基于Agnoster改造的，默认主题文件位于

IT之家学院：使用oh-my-posh美化Win10 PowerShell

接下来便是在PowerShell启动时加载这个模组了。类似于Linux Bash的.bashrc，PowerShell也提供类似的Profile文件用于在启动时执行指令。输入

```powershell
Test-Path $profile
```

并执行，以确定profile文件是否存在。如果返回False，则应该执行：

```powershell
New-Item -path $profile -type file–force
```

来新建一个文件。然后去往Profile的目录（通常是你的文档下的WindowsPowerShell目录下），修改那个后缀为ps1的Profile文件，加入一行Import-Module oh-my-posh即可。

一切完成之后，PowerShell应该比原先美观了不少，而且提示符的功能更强了。基于oh-my-posh框架，还能自己编写更多的主题。