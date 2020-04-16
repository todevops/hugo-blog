---
title: macOS 设置 launchpad 图标大小
date: 2019-10-15 01:31:26
categories:
  - mac
tags:
  - mac
---

KMacOS 设置 launchpad 图标大小
<!--more-->

```
defaults write com.apple.dock springboard-rows -int 7
defaults write com.apple.dock springboard-columns -int 6
killall Dock
defaults write com.apple.dock ResetLaunchPad -bool TRUE;killall Dock
```
