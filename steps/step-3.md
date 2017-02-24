## 设计 app shell 框架

### 什么是 app shell

app shell 其实是 PWA 用来启动 UI 的最小 HTML、CSS、JavaScript 集合，它是可靠地保证 app 良好性能的一个组件。首次加载非常快，且加载后会被立即缓存在本地设备上，后续启动直接从本地读取文件，速度超级快。

![app shell](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/156b5e3cc8373d55.png)

app shell 将核心应用架构和 UI 从数据中分离出来。所有的 UI 和架构都被 service worker 缓存在本地，PWA 只需要请求必要的数据。

换个思路，app shell 和发布到 app store 的 app 打包代码比较相似。

### 为何使用 app shell 框架

使用 app shell 可以帮助你更专注与速度，能为 PWA 提供类似原生 app 的相似属性。

### 设计 app shell

首先需要思考：

1. 什么需要立即在屏幕上显示出来？
1. 对于 app 来说哪些除此之外的组件是关键？
1. app shell 需要哪些资源支持？比如图片、js、样式等。

我们使用 Weather App 作为我们的第一个 PWA。其核心组件包括：

- 头部，包括标题、添加和刷新按钮
- 预告卡片的容器
- 预告卡片模版
- 添加新城市的对话框
- 加载图标

![Weather App](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/166c3b4982e4a0ad.png)

*注意：当设计的 app 更加复杂时，不是初始载入必须的内容可以稍后再请求，然后缓存起来供将来使用。例如，我们可以把添加新城市的对话框放到首次渲染之后有空闲时间的时候加载。*
