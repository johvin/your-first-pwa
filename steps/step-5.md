## 从第一次快速载入开始

PWA 应该快速启动并且立即可用。在现阶段，我们的 app 可以快速启动，但没有可用性，因为没有数据。我们可以发起 ajax 请求数据，但是会带来延迟。替代的解决方案是首次加载即提供真是数据。

### 注入天气预报数据

在我们的教程中，我们假设 server 直接想 js 中注入预报数据，但是在上线的 app，最新的预报数据是由 server 根据用户 IP 地址定位到用户所在位置进而注入的。

app.js 中已经包含要注入的数据了，即 `initialWeatherForecast`。

### 区分第一次运行

我们怎么确定什么时候应该显示注入的数据呢？该数据可能与将来从 cache 启动的 app 不相关。当用户后续访问 app，他们可能已经更改了城市。我们需要为这些城市载入数据，而不一定是他们曾经查找过的第一个城市。

用户的设定需要存储在本地，可以使用 IndexDB 或其他快速的存储机制。这里为了简化，我们使用 localStorage，但它并不是理想的线上解决方案，因为它是同步阻塞的存储机制，在某些设备上会很慢。

*可以使用 [`idb`](https://www.npmjs.com/package/idb) 代替 localStorage，[`localForage`](https://github.com/localForage/localForage) 是对 `idb` 的简单封装。*

下面我们来完成保存用户选择城市的函数功能，找到下面的 TODO 注释行：

```js
// TODO add saveSelectedCities function here
```

添加如下代码：

```js
// Save list of cities to localStorage.
  app.saveSelectedCities = function() {
    var selectedCities = JSON.stringify(app.selectedCities);
    localStorage.selectedCities = selectedCities;
  };
```

然后添加启动代码来检查用户是否设置来城市或者直接使用注入的数据。注释如下：

```js
// TODO add startup code here
```

替换如下：

```js
/************************************************************************
   *
   * Code required to start the app
   *
   * NOTE: To simplify this codelab, we've used localStorage.
   *   localStorage is a synchronous API and has serious performance
   *   implications. It should not be used in production applications!
   *   Instead, check out IDB (https://www.npmjs.com/package/idb) or
   *   SimpleDB (https://gist.github.com/inexorabletash/c8069c042b734519680c)
   ************************************************************************/

  app.selectedCities = localStorage.selectedCities;
  if (app.selectedCities) {
    app.selectedCities = JSON.parse(app.selectedCities);
    app.selectedCities.forEach(function(city) {
      app.getForecast(city.key, city.label);
    });
  } else {
    /* The user is using the app for the first time, or the user has not
     * saved any cities, so show the user some fake data. A real app in this
     * scenario could guess the user's location via IP lookup and then inject
     * that data into the page.
     */
    app.updateForecastCard(initialWeatherForecast);
    app.selectedCities = [
      {key: initialWeatherForecast.key, label: initialWeatherForecast.label}
    ];
    app.saveSelectedCities();
  }
```

### 保存选择的城市

更改按钮 `butAddCity` 的点击处理函数，修改如下：

```js
document.getElementById('butAddCity').addEventListener('click', function() {
    // Add the newly selected city
    var select = document.getElementById('selectCityToAdd');
    var selected = select.options[select.selectedIndex];
    var key = selected.value;
    var label = selected.textContent;
    if (!app.selectedCities) {
      app.selectedCities = [];
    }
    app.getForecast(key, label);
    app.selectedCities.push({key: key, label: label});
    app.saveSelectedCities();
    app.toggleAddDialog(false);
  });
```

### 试试看

- 首次启动，app 应该立即显示 `initialWeatherForecast` 的预报数据。
- 点击添加城市按钮，验证是否有两个预告卡片显示出来。
- 刷新页面，验证是否显示两个城市的预报并且是最新信息。
