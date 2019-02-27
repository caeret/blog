---
title: "使用 DevDocs 和 HammerSpoon 替代 Dash"
date: 2017-12-21T17:59:47+08:00
categories: ["macOS"]
tags: ["Dash", "DevDocs", "HammerSpoon"]
---

作为一个码农，平时做开发，查看 API 文档是一个高频动作。在 MacOS 平台，有个查询 API 的神器，即 Dash。然而 Dash 是需要收费的，如果未付费使用的话，会在启动时需要等待一定的时间才可使用。如果觉得不方便，还有个替代方案，那就是 Devdocs。使用了一下对比后发现，相对 Dash，DevDocs 会对文档排版做一定的重新解析，看起来更清晰统一，而不像 Dash 那样看着像是直接从官网缓存页面到本地的。

在使用 DevDocs 时，如果你不想每次都要打开浏览器然后才能查文档，可以使用 electron 封装的版本来达到类似桌面应用的体验。比如：https://github.com/egoist/devdocs-desktop

<!--more-->

## 利用 HammerSpoon 配置快捷键

在 Dash 中，可以使用 `command + '` 全局快捷键来激活和隐藏 Dash 窗口，可以在使用时，可以随机用快捷键激活 Dash 查看文档。

使用 electron 封装的桌面版时，可能没有提供类似这样的全局快捷键。这时候，HammerSpoon 就派上用场了。

安装了 HammerSpoon 后，可以在配置脚本中添加以下这段代码，来提供 `command + '` 全局启动、激活和隐藏 DevDocs 的目的。

```lua
-- 绑定 command 和 ' 键作为全局快捷键
hs.hotkey.bind("cmd", "'", function()
  -- 应用名为 DevDocs
  local name = "DevDocs"
  local app = hs.application.get(name)
  if app == nil then
    hs.application.launchOrFocus(name)
  elseif app:isFrontmost() then
    app:hide()
  else
    app:activate()
  end
end)
```

当然，你也可以根据自己的喜好，将全局快捷键配置到其他键位，只需要稍微修改下上面脚本就可以了。这样每次要查文档时，只需要快捷键就可以呼出窗口了。




























