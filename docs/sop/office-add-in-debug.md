---
title: 在 Mac 上调试 Office 加载项
date: 2023-04-13 21:46:24
tags: office加载项
categories: 前端
description: 由于加载项是使用 HTML 和 JavaScript 开发的，因此它们设计为跨平台工作，但不同浏览器呈现 HTML 的方式可能存在细微差异。 本文介绍如何调试在 Mac 上运行的加载项。
---

# 在 Mac 上调试 Office 加载项

由于加载项是使用 HTML 和 JavaScript 开发的，因此它们设计为跨平台工作，但不同浏览器呈现 HTML 的方式可能存在细微差异。 本文介绍如何调试在 Mac 上运行的加载项。

## 在 Mac 上使用 Safari Web 检查器进行调试

如果有在任务窗格或内容加载项中显示 UI 的加载项，可以使用 Safari Web 检查器调试 Office 加载项。

为了能够在 Mac 上调试 Office 加载项，必须具有 Mac OS High Sierra AND Mac Office 版本 16.9.1 (内部版本18012504) 或更高版本。 如果没有 Office on Mac 版本，可以通过加入 Microsoft 365 开发人员计划来获取一个。

首先，打开终端，设置相关 Office 应用程序的 OfficeWebAddinDeveloperExtras 属性，如下所示：

- `defaults write com.microsoft.Word OfficeWebAddinDeveloperExtras -bool true`
- `defaults write com.microsoft.Excel OfficeWebAddinDeveloperExtras -bool true`
- `defaults write com.microsoft.Powerpoint OfficeWebAddinDeveloperExtras -bool true`
- `defaults write com.microsoft.Outlook OfficeWebAddinDeveloperExtras -bool true`

> 注意：Mac App Store Office 版本不支持 标志OfficeWebAddinDeveloperExtras。

然后，打开 Office 应用程序并旁加载你的加载项。 右键单击加载项，应在上下文菜单中看到一个“检查元素”选项。 选择该选项，它将弹出检查器，可以在其中设置断点并调试加载项。

参考：[https://learn.microsoft.com/zh-cn/office/dev/add-ins/testing/debug-office-add-ins-on-ipad-and-mac](https://gitee.com/link?target=https%3A%2F%2Flearn.microsoft.com%2Fzh-cn%2Foffice%2Fdev%2Fadd-ins%2Ftesting%2Fdebug-office-add-ins-on-ipad-and-mac)