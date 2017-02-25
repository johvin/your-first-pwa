## 使用 service worker 预缓存 app shell

PWA 应用必须快速、可安装，这意味着它们在有网络、无网络、网络断断续续、网速慢的环境都能工作。要达到这个目标，我们需要使用 service worker 来缓存 app shell，这样它总能快速可靠的可用。

如果你对 service worker 还不熟悉，可以阅读 [Service Worker 介绍](https://developers.google.com/web/fundamentals/primers/service-worker/) 来初步了解它能做什么，它的生命周期以及更多内容。一旦你完成本教程的内容，一定要学习 [调试 Service Worker](https://codelabs.developers.google.com/codelabs/debugging-service-workers/#0) 来更深入的学习如何使用 service worker 来开发。

service worker 提供的特性应该被认为是渐进增强，只有在浏览器支持的情况下才能使用。例如，service worker 可以缓存 app shell 和数据，在离线的时候使用。当 service worker 不被支持的环境，离线代码不会被调用，用户可以得到一个基础体验。使用特性检测来提供渐进增强功能代价很小并且在不支持的旧版本浏览器上也不会影响页面的功能。

*service worker 的功能支持 https 的页面上可用（ http://localhost 和等价的网址也能工作，仅仅是为了测试）。想学习这个限制背后的原理，可以从 Chromium team 查看 [Prefer Secure Origins For Powerful New Features](http://www.chromium.org/Home/chromium-security/prefer-secure-origins-for-powerful-new-features) 这篇文章。*

### 如果可用则注册 Service Worker

要让 app 在离线环境下工作的第一步是注册 service worker，它是一个不需要打开页面或用户交互的允许在后台运行的脚本。

注册分两步：

1. 告诉浏览器将一个 JavaScript 文件注册为 service worker
1. 创建 service worker 脚本文件

代码如下：

```js
if ( 'serviceWorker' in navigator ) {
  navigator.serviceWorker.register('/service-worker.js')
  .then(function (event) {
    console.log('Service Worker Registered!');
  });
}
```

### 缓存站点资源

当 service worker 被注册，install 事件会在用户第一次访问页面当时候被触发。在该事件处理函数中，我们会缓存 app 需要的所有资源。

*注意，下面的代码不可以用在线上，因为它只包含来基本的示例，会导致 app shell 永远等不到更新。*

当 service worker 启动后，应该打开 caches 对象，并将要缓存的资源添加进去。在应用根目录创建 service-worker.js 文件，因为在上面的注册的代码中我们制定了它的位置。把下面的代码添加到文件中：

```js
var cacheName = 'weatherPWA-step-6-1';
var filesToCache = [];

self.addEventListener('install', function(e) {
  console.log('[ServiceWorker] Install');
  e.waitUntil(
    caches.open(cacheName).then(function(cache) {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll(filesToCache);
    })
  );
});
```

代码的意思就不解释了，这里要注意，`cache.addAll` 是原子性的，如果任何一个文件请求失败，那么整个缓存都会失败。

下面简单介绍下怎么使用 DevTools 来调试 service worker。刷新页面前，先打开开发者工具，切换到 Application 面板中的 Service Worker 面板。如下图：

![Service Worker Pane empty](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/ed4633f91ec1389f.png)

面板是空的表示当前页面还没有注册 service worker。然后刷新页面，Service Worker 面板会发生变化：

![Service Worker Pane](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/bf15c2f18d7f945c.png)

如果你的面上变成这样，说明有一个 service worker 正在运行。

现在，我们来举个例子来解释开发过程中会遇到的问题。在此之前，我们要在 install 事件后面添加 activate 事件处理函数，如下：

```js
self.addEventListener('activiate', function (event) {
  console.log('[ServiceWorker] Activate');
});
```

service worker 启动时会出发 activate 事件。

打开控制台，刷新页面，打开 Service Worker 面板在已激活的 service worker 上点击 inspect。期望的效果是 log 输出到控制台，但实际上并没有。切换到 Service Worker 面板，你会看到新的 service worker （包括已激活事件处理函数）处于等待的状态。效果如下：

![sw wait status](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/1f454b6807700695.png)

基本上，如果有一个 tab 页打开了当前页面，那么旧的 service worker 就会一直控制着页面。因此，你需要关闭再打开当前页面或者点击 `skipWaiting` 按钮，但更好的方法是启用 `Update on Reload`。启用后，每次页面刷新都会强制更新 service worker。

勾选 `Update on Reload` 并刷新页面，确保新 service worker 被激活。

以上就是 service worker 的检查和调试的内容，后面会有更多内容。先让我们回到构建 app 的流程中。

我们展开 activiate 事件处理函数，里面包含一些更新 cache 的逻辑。更新后的代码如下：

```js
self.addEventListener('activiate', function (event) {
  console.log('[Service Worker] Activiate');
  event.waitUntil(
    caches.keys().then(function (keyList) {
      return Promise.all(keyList.map(function (key) {
        if (key !== cacheName) {
          console.log('[Service Worker] removing old cache', key);
          return caches.delete(key);
        }
      }));
    });
  );
  return self.clients.claim();
});
```

这段代码确保 app shell 的文件更改时，service worker 可以及时更新 cache。为了实现这个功能，你需要将 cacheName 变量提升到 service worker 文件的顶部。

最后一句修复了一种边角情况，具体可以看下面的解释。

*当 app 准备完全后，`self.clients.claim()` 会修复“app 不返回最新数据”的边角情况。这种情况可以复现，注释掉这行代码后进行如下操作：1）首次加载 app，显示纽约市的数据。2）刷新页面。3）切换成离线。4）重新加载 app。 期望的效果是显示纽约新的数据，但实际看到的是初始化的数据。这种情况出现的原因是 service worker 还没有激活。`self.clients.claim()` 会让你更快的激活 service worker。(译者注：这个例子并不能验证 `self.clients.claim()` 的作用，因为目前为止，应用并没有缓存预报数据，因此再次刷新后新数据就会丢失，只会显示初始化的数据。这个实验可以在完成下一节 “使用 service worker 缓存预报数据” 之后再做一次，应该就会得到想要的结果。)*

最后，我们来更新下 app shell 需要缓存的资源，如下：

```js
var filesToCache = [
  '/',
  '/index.html',
  '/scripts/app.js',
  '/styles/inline.css',
  '/images/clear.png',
  '/images/cloudy-scattered-showers.png',
  '/images/cloudy.png',
  '/images/fog.png',
  '/images/ic_add_white_24px.svg',
  '/images/ic_refresh_white_24px.svg',
  '/images/partly-cloudy.png',
  '/images/rain.png',
  '/images/scattered-showers.png',
  '/images/sleet.png',
  '/images/snow.png',
  '/images/thunderstorm.png',
  '/images/wind.png'
];
```

确保包含来所有的文件名的组合。例如我们的 app 的地址是 `index.html`，但它也可以通过请求 `/` 被请求到。你可以在 fetch 中处理这种请求，但那样会需要添加特殊的判断逻辑，会更复杂。

### 从缓存响应 app shell

Service Worker 提供了拦截 PWA 网络请求并且在 service worker 中处理的能力。这意味着我们可以决定要怎样处理请求。

例如：

```js
self.addEventListener('fetch', function (e) {
  // Do something interesting with the fetch here
});
```

对代码做如下更改以支持从缓存响应 app shell。

```js
self.addEventListener('fetch', function (e) {
  console.log('[Service Worker] Fetch', e.request.url);
  e.respondWith(
    caches.match(e.request).then(function (response) {
      return response || fetch(e.request);
    });
  );
});
```

*如果在控制台没有看到期望的 log，确保更改了 cacheName 并且查看的是对的 service worker，并且在对的 service worker 上点击 inspect 连接。如果还是不正常，请查看后面 [测试 service worker 的建议](#测试-service-worker-的建议) 一节的内容。*

### 验证

刷新页面，查看 Application 面板下的 Cache 面板，你会看到如下内容：

![cache page](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/ab9c361527825fac.png)

下面我们来测试离线模式。在 Service Worker 面板勾选 offline，如下：

![offline settings](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/7656372ff6c6a0f7.png)

此时 Network 面板会出现一个黄色图标，代表网络不可用。刷新页面，app 可以正常工作。注意初始数据（假的）是怎么加载的。这需要查看 `app.getForecast()` 的 else 分之代码。

### 了解边界情况

像前面提到的，目前的代码不要应用在线上，因为还有很多情况没有处理。这里只列出所有的边界情况。

- 缓存是基于每次更改都要更新 cache 键值
- 每次更改都需要全部资源下载完毕
- 浏览器缓存可能会阻止 service worker 缓存更新
    + install 事件处理函数中发出的 https 请求能直达网络，返回的响应不是浏览器缓存的结果。其它情况浏览器可能会返回旧的、缓存的版本，导致 service worker 的缓存永远也无法更新。
- 意识到线上的缓存优先策略
    + 缓存优先策略会导致不请求网络就直接返回缓存的内容。问题也接踵而来。一旦页面和 service worker 注册被缓存，就很难改变 service worker 的配置了（因为配置是依赖它定义的地方），并且你会发现发布的站点很难去更新。
- 怎么避免这些边界情况
    + 使用像 [sw-precache](https://github.com/GoogleChrome/sw-precache) 这样的库，它提供了根据文件过期时间的缓存控制，确保请求能直接到达网络并且为你把困难的工作都做了。

### 测试 service worker 的建议

调试 service worker 是一个很大的挑战，当缓存没有按照预期更新，事情会变成噩梦一般。你会疲于本命于典型的 service worker 生命周期和代码 bug 之间。不过别灰心，这里有一些工具可以帮到你：

#### 重新开始

有时候，你发现应用正在使用缓存的数据或者应用的行为表现的不像预期一样。这时候应该清空所有保存的数据（localStorage、indexDB、缓存的文件）并且移除页面中所有的 service worker。你可以使用 `Clear storage` 面板来简化操作。

一些其它建议：

- 一旦 service worker 被注销，它可能在窗口被关闭前一直被列出来。
- 如果多个标签页打开同一个窗口，那么新 service worker 在没有被重新加载并更新到最新 service worker 之前都不会生效。
- 注销 service worker 不会清空缓存，所以 cacheName 没有改变到话，你可能一直在使用旧数据。
- 如果一个 service worker 存在，另一个新的 service worker 被注册，新 service worker 直到页面重新加载或者采取了[立即控制](https://github.com/GoogleChrome/samples/blob/gh-pages/service-worker/immediate-control/service-worker.js)措施才能接管控制权。
