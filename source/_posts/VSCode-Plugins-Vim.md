---
title: VS Code 插件篇 - Vim 与 Windows 快捷键
date: 2016-12-20 16:32:50
tags: 
    - VS Code
---
Vim 不仅仅是一个高效的文本编辑器，更是一套优秀的操作方式。在 VS Code 中我们可以通过安装插件的方式使用 Vim 模式以提高工作效率。

![](http://image.grimoire.cc/blog/png/vscode-vim-1.png)

但作为刚刚开始学习这套操作方式的小白发现，切换 Vim 模式后，连常用的 Ctrl+C Ctrl+V Ctrl+D 也无法正常使用了。那么有没有什么方法能够既不抛弃常用的 Windows 按键，同时还能使我们慢慢熟悉 Vim 的操作模式呢？

## 修改配置文件
VS Code 插件配置文件地址：`C:\Users\用户名\.vscode\extensions\`

我们需要在这其中找到 Vim 插件的配置文件进行编辑

例：`C:\Users\dell\.vscode\extensions\vscodevim.vim-0.4.10\package.json`

![](http://image.grimoire.cc/blog/png/vscode-vim-2.png)

然后找到
```
    "vim.useCtrlKeys": {
        "type": "boolean",
        "description": "Enable some vim ctrl key commands that override otherwise common operations, like ctrl+c",
        "default": true
    },
```
将 true 修改为 false 保存即可。

![](http://image.grimoire.cc/blog/png/vscode-vim-3.png)

这样我们的 Windows 常用快捷键就回来了，可以边学习 Vim 边混合使用了。

