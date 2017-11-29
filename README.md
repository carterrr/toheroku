#heroku部署nodejs爬虫项目

## 目标

建立一个 项目，在其中编写代码并发布到heroku。

在命令行输入heroku open 时，浏览器页面输出 CNode(https://cnodejs.org/ ) 社区首页的所有帖子标题和链接，以 json 的形式。

输出示例：

```js
[
  {
    "title": "【公告】发招聘帖的同学留意一下这里",
    "href": "http://cnodejs.org/topic/541ed2d05e28155f24676a12"
  },
  {
    "title": "发布一款 Sublime Text 下的 JavaScript 语法高亮插件",
    "href": "http://cnodejs.org/topic/54207e2efffeb6de3d61f68f"
  }
]

```

## 挑战

输入 heroku open 时，输出包括主题的作者，

示例：

```js
[
  {
    "title": "【公告】发招聘帖的同学留意一下这里",
    "href": "http://cnodejs.org/topic/541ed2d05e28155f24676a12",
    "author": "alsotang"
  },
  {
    "title": "发布一款 Sublime Text 下的 JavaScript 语法高亮插件",
    "href": "http://cnodejs.org/topic/54207e2efffeb6de3d61f68f",
    "author": "otheruser"
  }
]
```

## 知识点

1. 学习使用 superagent 抓取网页
2. 学习使用 cheerio 分析网页
3. 学习项目部署

## 课程内容

Node.js 总是吹牛逼说自己异步特性多么多么厉害，但是对于初学者来说，要找一个能好好利用异步的场景不容易。我想来想去，爬虫的场景就比较适合，没事就异步并发地爬几个网站玩玩。


我们这回需要用到三个依赖，分别是 express，superagent 和 cheerio。

先介绍一下，

superagent(http://visionmedia.github.io/superagent/ ) 是个 http 方面的库，可以发起 get 或 post 请求。

cheerio(https://github.com/cheeriojs/cheerio ) 大家可以理解成一个 Node.js 版的 jquery，用来从网页中以 css selector 取数据，使用方式跟 jquery 一样一样的。

还记得我们怎么新建一个项目吗？

1. 新建一个文件夹，进去之后 `npm init`
1. 安装依赖 `npm install --save PACKAGE_NAME`
1. 写应用逻辑

我们应用的核心逻辑长这样

```js
app.get('/', function (req, res, next) {
  // 用 superagent 去抓取 https://cnodejs.org/ 的内容
  superagent.get('https://cnodejs.org/')
    .end(function (err, sres) {
      // 常规的错误处理
      if (err) {
        return next(err);
      }
      // sres.text 里面存储着网页的 html 内容，将它传给 cheerio.load 之后
      // 就可以得到一个实现了 jquery 接口的变量，我们习惯性地将它命名为 `$`
      // 剩下就都是 jquery 的内容了
      var $ = cheerio.load(sres.text);
      var items = [];
      $('#topic_list .topic_title').each(function (idx, element) {
        var $element = $(element);
        items.push({
          title: $element.attr('title'),
          href: $element.attr('href')
        });
      });

      res.send(items);
    });
});
```

OK，一个简单的爬虫就是这么简单。这里我们还没有利用到 Node.js 的异步并发特性。不过下两章内容都是关于异步控制的。

记得好好看看 superagent 的 API，它把链式调用的风格玩到了极致。
大家有没有想过，当部署一个应用上 paas 平台以后，paas 要为我们干些什么？

首先，平台要有我们语言的运行时；

然后，对于 node.js 来说，它要帮我们安装 package.json 里面的依赖；

然后呢？然后需要启动我们的项目；

然后把外界的流量导入我们的项目，让我们的项目提供服务。

上面那两处特殊的地方，一个是启动项目的，一个是导流量的。

heroku 虽然能推测出你的应用是 node.js 应用，但它不懂你的主程序是哪个，所以我们提供了 Procfile 来指导它启动我们的程序。

而我们的程序，本来是监听 5000 端口的，但是 heroku 并不知道。当然，你也可以在 Procfile 中告诉 heroku，可如果大家都监听 5000 端口，这时候不就有冲突了吗？所以这个地方，heroku 使用了主动的策略，主动提供一个环境变量 process.env.PORT 来供我们监听。

这样的话，一个简单 app 的配置就完成了。

我们去 https://www.heroku.com/ 申请个账号，然后下载它的工具包 https://toolbelt.heroku.com/ ，然后再在命令行里面，通过 heroku login 来登录。

上述步骤完成后，我们进入 node-practice-2 的目录，执行 heroku create。这时候，heroku 会为我们随机取一个应用名字，并提供一个 git 仓库给我们。



接着，往 heroku 这个远端地址推送我们的 master 分支：



heroku 会自动检测出我们是 node.js 程序，并安装依赖，然后按照 Procfile 进行启动。

push 完成后，在命令键入 heroku open，则 heroku 会自动打开浏览器带我们去到相应的网址：




随便聊聊 heroku 的 addon 吧。这个 addon 确实是个神奇的东西，反正在 heroku 之外我还没怎么见到这类概念。这些 addon 提供商，有些提供 redis 的服务，有些提供 mongodb，有些提供 mysql。你可以直接在 heroku 上面进行购买，然后 heroku 就会提供一段相应服务的地址和账号密码给你用来连接。

大家可以去 https://addons.heroku.com/ 这个页面看看，玲琅满目各种应用可以方便接入。之所以这类服务有市场，也是因为亚马逊的 aws 非常牛逼。为什么这么说呢，因为网络速度啊。如果现在在国内，你在 ucloud 买个主机，然后用个阿里云的 rds，那么应用的响应速度会因为 mysql 连接的问题卡得动不了。但在 heroku 这里，提供商们，包括 heroku 自己，都是构建在 aws 上面，这样一来，各种服务的互通其实走的是内网，速度很可以接受，于是各种 addon 提供商就做起来了。

国内的话，其实在阿里云上面也可以考虑这么搞一搞。

完。
