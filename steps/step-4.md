## 实现 app shell

开始一个新项目的方法很多，我们推荐 [Web Starter Kit](https://developers.google.com/web/tools/starter-kit/)。在这个项目中，为了保持项目的简单，我们把所有的素材都准备好了。

### 创建 html 的 app shell

根据上一节的需要完成的组件清单，我们完成了 work 目录下的 `index.html`，这里将其做了简化列出来。

```HTML
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weather PWA</title>
  <link rel="stylesheet" type="text/css" href="styles/inline.css">
</head>
<body>
  <header class="header">
    <h1 class="header__title">Weather PWA</h1>
    <button id="butRefresh" class="headerButton"></button>
    <button id="butAdd" class="headerButton"></button>
  </header>

  <main class="main">
    <div class="card cardTemplate weather-forecast" hidden>
    . . .
    </div>
  </main>

  <div class="dialog-container">
  . . .
  </div>

  <div class="loader">
    <svg viewBox="0 0 32 32" width="32" height="32">
      <circle id="spinner" cx="16" cy="16" r="14" fill="none"></circle>
    </svg>
  </div>

  <!-- Insert link to app.js here -->
</body>
</html>
```

*注意：加载图标默认显示出来，这样可以保证页面一加载出来用户就能看到加载图标，能够第一时间告诉用户页面正在加载内容。*

### 检出 js 代码

work 目录下包含了 js 代码 `scripts/app.js`，其包含的内容有：

- 包含应用关键信息的 app 对象
- header 部分和添加城市对话框中的所有按钮的事件监听器
- 添加或更新预告卡片的方法
- 从 firebase public weather api 获取最新的数据
- 遍历所有预告卡片并获取最新数据
- 测试渲染机制能否正常工作的假数据

### 试一试

去掉 html 中的下列代码的注释：

```html
<!--<script src="scripts/app.js" async></script>-->
```

以及去掉 app.js 底部的这一行代码的注释：

```js
// app.updateForecastCard(initialWeatherForecast);
```

期望的运行结果是这样的：

![fake data](https://codelabs.developers.google.com/codelabs/your-first-pwapp/img/166c3b4982e4a0ad.png)
