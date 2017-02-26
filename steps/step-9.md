## 分发到安全的主机然后庆祝一下

最后一步是将你的 app 分发到一个支持 https 的服务器上。如果你还没有这样的服务器，那么绝对最简单（并且免费的）方法是使用 Firebase 提供的静态内容主机服务。它使用起来超级简单，通过 https 提供内容服务，并且有全球 CDN 服务支持。

### 额外加分项：压缩和内联 CSS

还有个事情你应该考虑一下，最小化关键样式并把它们直接内联到 `index.html` 中。[PageSpeed Insights](https://developers.google.com/speed) 推荐在请求的最初 15K 字节中提供上述折叠内容。

看看将所有内容都内联后初始请求会变成多小。

深入了解：[PageSpeed Insight 规则](https://developers.google.com/speed/docs/insights/rules)。

*这一步要求你在系统中安装 [Node & NPM](https://docs.npmjs.com/getting-started/installing-node)。如果没有安装，你可以选择任何其它支持 https 的服务托管提供商。我们使用 Firebase 因为它能自动将用户从 HTTP 重定向到 HTTPS。如果你有其它提供商，确保它们总是会重定向到 HTTPS。*

### 分发到 Firebase

如果你对 Firebase 还比较陌生，你需要先创建你的账号并安装一些工具。

1. 在 [https://firebase.google.com/console/](https://firebase.google.com/console/) 创建一个 Firebase 账号。
1. 通过 npm 命令 `npm install -g firebase-tools` 安装 Firebase 工具。

一旦你的账号创建完毕并已经登录，分发的准备就完成了。

1. 在 [https://firebase.google.com/console/](https://firebase.google.com/console/) 创建一个新 app
1. 如果你最近还没有登录 Firebase 工具，更新你的凭据／证书：`firebase login`
1. 初始化你的 app，提供已完成的 app 所在的目录（比如 `work`)：`firebase init`
1. 最后将 app 分发到 Firebase：`firebase deploy`
1. 庆祝。你完成了！你的 app 会被分发到这个域：`https://YOUR-FIREBASE-APP.firebaseapp.com`

深入了解：[Firebase 托管指南](https://www.firebase.com/docs/hosting/guide/)

### 试试看

试着将 app 添加到主屏，然后断开网络验证 app 是否像预期一样在离线模式下正常工作。
