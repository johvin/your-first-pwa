## 使用 service worker 缓存预报数据

选择正确的[缓存策略](https://jakearchibald.com/2014/offline-cookbook/)是很重要的，选择依赖于你的应用要展示数据的类型。比如，像天气和股票报价一类的时效性数据应该尽可能的保持最新，而头像图片或文章内容更新的频率可以低一些。

[缓存第一网络第二](https://jakearchibald.com/2014/offline-cookbook/#cache-then-network)（译者改：此处原文的链接地址是 [cache and network race](https://jakearchibald.com/2014/offline-cookbook/#cache-network-race)，根据后面的解释可以判断文章的意思应该是改后的链接内容）是我们 app 理想的缓存策略。这个策略会尽快的将数据显示出来，一旦网络返回最新数据，页面也会更新。与[网络第一缓存第二](https://jakearchibald.com/2014/offline-cookbook/#network-falling-back-to-cache)策略相比，用户不必等待网络请求超时就可以直接看到内容。

`缓存第一网络第二` 意味着我们需要发起两个异步请求，一个请求缓存，一个请求网络。网络请求的代码不需要太多更改，但是我们需要更改 service worker 的代码以便在返回网络请求的结果前缓存下来。

通常情况下，缓存的数据会几乎立即将它可用的最新数据返回给页面，然后当网络请求得到响应后，app 会使用从网络拿到的最新数据更新页面。

### 拦截网络请求并缓存响应的数据

我们需要修改 service worker 以便拦截对天气 API 的请求并且把返回的响应存储在缓存中，因此我们之后可以更容易的访问。对于 `缓存第一网络第二` 策略，我们期望网络响应是预报数据的来源，永远为我们提供最新的信息。如果网络不能提供，那么优雅降级也是可以的，因为我们已经从最新的缓存数据拿到数据了。

在 service worker 中，让我们来添加一个 `dataCacheName` 以便我们可以将数据和 app shell 分开。当 app shell 被更新并且旧的缓存被清除，我们的数据仍然没有被更新，准备为应用提供一次超级快速的启动。请记住，如果将来数据格式发生变化，你会需要一种途径来处理它并确保 app shell 和内容保持同步。

在 service-worker.js 头部添加下面一行

```js
var dataCacheName = 'weatherData-1';
```

然后更新 `activate` 事件处理函数的逻辑以便在清除 app shell 缓存时不要删除数据缓存。

```js
if (key !== cacheName && key !== dataCacheName) {
```

最后更新 `fetch` 事件处理器功能来分开处理发给数据 API 的请求和其它的请求。

```js
self.addEventListener('fetch', function (e) {
  console.log('[Service Worker] fetch', e.request.url);
  var dataUrl = 'https://query.yahooapis.com/v1/public/yql';
  if (e.request.url.indexOf(dataUrl) > -1) {
    /*
     * 当请求 url 包括 dataUrl 时，app 是在请求新鲜的天气数据。
     * 这种情况下，service worker 要去网络请求数据并缓存响应结果。
     * 这就是”缓存第一网络第二”策略：
     * https://jakearchibald.com/2014/offline-cookbook/#cache-then-network
     */
    e.respondWith(
      caches.open(cacheName).then(function (cache) {
        return fetch(e.request).then(function (response) {
          cache.put(e.request.url, response.clone());
          return response;
        });
      });
    );
  } else {
    /*
     * app 请求 app shell 文件。在这个场景下，app 使用“缓存优先、降级到网络” 的离线策略：
     * https://jakearchibald.com/2014/offline-cookbook/#cache-falling-back-to-network
     */
    e.respondWith(
      caches.match(e.request).then(function (response) {
        return response || fetch(e.request);
      });
    );
  }
});
```

上面代码的意思相信大家都能理解，这里就不解释了。

现在，我们的 app 还不能在离线模式下工作。我们实现了缓存、获取 app shell 文件的功能，但即使我们缓存了数据，app 依然没有检查缓存来查看是否有可用的天气数据。

### 发起请求（译者注：发起检查数据缓存的请求）

就像上面提到的，app 应该发起两个异步请求。一个到 cache，一个到网络。app 使用 `window` 可见的 `caches` 对象来访问缓存并获取最新的数据。这是一个渐进增强的非常好的例子，因为 `caches` 对象不是在所有浏览器里都可用的，在不可用的浏览器中，网络请求依然能正常工作。

这里不给出详细实现步骤，直接给出代码实现。（原文此处有具体步骤，因为看代码就一目了然，所以此处省略。）

#### 从缓存获取数据

接下来，我们需要检查 `caches` 对象是否存在并从它获取最新的数据。找到 `app.getForecast()` 中的 `TODO add cache logic here` 这行注释，然后在下面添加如下代码：

```js
if ('caches' in window) {
  caches.match(url).then(function (response) {
    if (response) {
      response.json().then(function updateFormCache(json) {
        var results = json.query.results;
        results.key = key;
        results.label = label;
        results.created = json.query.created;
        app.updateForcastCard(results);
      });
    }
  });
}
```

我们的天气 app 现在发出里两个异步请求获取预报数据，一个到 cache，一个经过 XHR。如果 cache 中有数据，它会立即返回并非常快的渲染（几十毫秒），前提是 XHR 尚未返回结果。然后，当 XHR 返回结果后，预报卡片会使用最新数据再次被更新。

*注意 cache 请求和 XHR 请求的结尾都会调用方法更新预报卡片数据。那 app 是怎么知道正在显示的是最新的数据呢？这个处理逻辑在 `app.updateForecastCard()` 中的下面代码中：*

```js
var cardLastUpdateElem = card.querySelector('.card-last-updated');
var cardLastUpdated = cardLastUpdateElem.textContent;
if (cardLastUpdated) {
  cardLastUpdated = new Date(cardLastUpdated);
  // Bail if the card has more recent data then the data
  if (dataLastUpdated.getTime() < cardLastUpdated.getTime()) {
    return;
  }
}
```

每次卡片更新，app 都会将数据的时间戳存储在 card 的一个隐藏属性上。如果时间戳存在且比传进来的数据更新，那么更新过程就结束。

### 试试看

app 现在应该完全支持离线功能。保存一些城市，点击 app 上的刷新按钮获取最新数据，然后转为离线模式再刷新页面。

然后打开 Application 面板下的 Cache Storage 面板，展开左侧区域能看到 app shell 和数据的缓存。数据缓存下能看到每个城市数据的缓存。

![Cache Storage](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/cf095c2153306fa7.png)
