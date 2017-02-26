## 支持原生集成

没人愿意在他们不需要的时候使用移动设备上的键盘键入一个很长的 url。有了添加到主屏的特性，你的用户可以选择添加一个快捷链接到设备上，就像他们从 app store 安装一个 app 一样，只是少了很多麻烦的操作。

### Android 版 Chrome 的 Web app 安装横幅以及添加到主屏功能

Web app 安装横幅赋予你让你的用户可以快速、无缝的将 web app 添加到他们设备的主屏上的能力，从而使得启动 app 和返回 app 变得很简单。添加 app 安装横幅是很简单的，Chrome 已经为你做了绝大多数复杂的工作。我们只需要包含一个 web app 清单文件用来描述 app 的详细信息。

然后 Chrome 使用包括 service worker、SSL 状态以及访问频率启发式规则等一系列条件来决定什么时候显示安装横幅。另外，用户也可以使用 Chrome 中的“添加到主屏”菜单按钮来手动添加到主屏。

#### 用 `manifest.json` 文件来声明一个 app 的清单

web app 清单是一个简单的 JSON 文件，为开发者提供了控制 app 在用户希望看到应用的区域中（例如移动设备的主屏）的展示方式的能力，指导用户可以启动什么，更重要的是如何启动它。

使用 web app 清单，你的 web app 能够：

- 在用户 Android 设备主屏以丰富的形式存在
- 在 Android 上以全屏且不带地址栏的模式启动
- 控制屏幕旋转以实现最理想的视觉效果
- 为站点定义“第一屏”的启动体验和主题颜色
- 跟踪应用是从主屏启动还是从地址栏启动的

在 `work` 目录下创建一个 `manifest.json` 文件，并复制黏贴下面的内容：

```json
{
  "name": "Weather",
  "short_name": "Weather",
  "icons": [{
    "src": "images/icons/icon-128x128.png",
    "sizes": "128x128",
    "type": "image/png"
    }, {
    "src": "images/icons/icon-144x144.png",
    "sizes": "144x144",
    "type": "image/png"
    }, {
      "src": "images/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }, {
      "src": "images/icons/icon-256x256.png",
      "sizes": "256x256",
      "type": "image/png"
    }],
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#3E4EB8",
  "theme_color": "#2F3BA2"
}
```

清单支持 icons 数组，适用于不同的屏幕尺寸。在写作本文的时候，唯一支持 web app 清单的浏览器 Chrome 和 Opera Mobile 不可以使用小于 192x192 尺寸的图片（译者注：根据下文 [使用 app 安装横幅](https://developers.google.com/web/fundamentals/engage-and-retain/simplified-app-installs/) 这篇文章的内容，icon 的尺寸不应小于 144x144，而不是这里的 192x192，Lighthouse 的报告也证明了这一点）。

追踪 app 启动方式的一个简单办法是添加一个查询字符串到变量 `start_url`，然后使用分析套件跟踪查询字符串。如果使用这个方法，*记得更新 app shell 的缓存文件列表，保证带有查询字符串的启动路径也被缓存了。*

#### 告诉浏览器你的清单文件

将下面的代码添加到 `index.html` 文件 `head` 标签的尾部：

```html
<link rel="manifest" href="/manifest.json" />
```

#### 最佳实践

- 将 manifest link 放到站点所有页面中，这样不论用户第一次访问的是哪个页面，Chrome 都能检索到这个信息。
- Chrome 上首选使用 `short_name`，如果在名字字段中显示，将使用 `short_name`。
- 为不同密度的屏幕定义图标集合。Chrome 会尝试使用最接近 48dp 的图标，例如，2 倍密度的设备是 96px，3 倍密度的设备是 144px。
- 记住，要包括一个大小对启动屏敏感的图标而且不能忘记设置 `background_color`。

深入阅读：[使用 app 安装横幅](https://developers.google.com/web/fundamentals/engage-and-retain/simplified-app-installs/)

### 为 iOS 版 Safari 增加添加到主屏功能的 html 元素

在 `index.html` 中的 `head` 元素末尾添加下面的代码：

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<meta name="apple-mobile-web-app-title" content="Weather PWA" />
<link rel="apple-touch-icon" href="images/icons/icon-152x152.png" />
```

### windows 的平铺图标

在 `index.html` 中的 `head` 元素末尾添加下面的代码：

```html
<meta name="msapplication-TileImage" content="images/icons/icon-144x144.png" />
<meta name="msapplication-TileColor" content="#2F3BA2" />
```

### 试试看

这一章节，我们会向你展示几种测试 web app 清单的方法。

第一个方法是就是开发者工具。打开 Application 面板下的 Manifest 面板。如果你正确的添加了清单信息，你就能在这个面板中看到被转换成人性化格式的清单信息了。

你也可以测试一下这个面板中“添加到主屏幕”的特性。点击“添加到主屏幕”按钮，你会在地址栏下方看到一个“添加到你的货架”的信息，如下图所示。

![Add to Home Screen](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/cbfdd0302b611ab0.png)

这就是桌面版中对应移动版的添加到主屏特性。如果你能在桌面版中成功触发此提示，那么你也可以确定移动用户也可以把你的 app 添加到他们的设备上。

第二种测试方式是通过 Web Server for Chrome。用这种方式，你可以将本地的开发服务器暴露给网络上的其它电脑，然后你可以通过真实设备访问你的 PWA 应用。

*为远程访问测试开启一个端口是很方便的，但有可能被你本机的防火墙或网络管理员限制。一直在你的电脑上开启远程端口访问并不好。因此，为了安全方面的原因，当你完成测试这个步骤后，请关闭“本地网络可访问”的选项并重启服务。*

在 Web Server for Chrome 的配置弹窗上，勾选“本地网络可访问”选项，如下图：

![Web Server configuration](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/81347b12f83e4291.png)

关闭 Web Server 再开启，你会看到一个新的 URL，它可以用来远程访问你的 app。

现在，使用新的 URL 从你的设备访问你的站点吧。

当你按照上面的方法开始测试的时候，你会在控制台看到 service worker 报错，因为 service worker 不是来自 https 服务。

使用 Android 设备上的 Chrome 试着将 app 添加到主屏幕上并验证启动屏和图标的正确性。

Safari 和 IE 上，你也可以手动将 app 添加到主屏幕。
