## 前言 ##

 大佬前辈要我做个Hubot自动问答机器人，要求平台在微信上，并且有较好的可扩展性，未来能和OA系统对接，更后期可能要求实现自动运维机器人。现阶段要求较低，需实现基本的聊天对话，查找一些内容等。

### 关于Hubot ###

 Hubot是GitHub出品的运维机器人，官方介绍[https://hubot.github.com/](https://hubot.github.com/ "HUBOT")，包含了官方的文档和很多开源的代码，是个好东西，想深入开发hubot要好好读一下文档以及各种内置函数接口等等。

### 安装Hubot ###

 Hubot是基于node.js体系的，使用coffeescript开发，node安装不多赘述，教程很多，只需要安装nodejs稳定版本即可，Windows下自带npm。这份文档是基于Windows系统的。

 官方推荐使用`yeoman + generator + hubot`生成器来生成hubot，所以我们先安装generator生成器，命令行中输入：

 	npm install -g yo generator-hubot
 	md testhubot
 	cd testhubot
 	npm install yo generator-hubot

 第一遍是全局安装，第二次是在本目录中安装，我一般都是全局和本地都有一份，接下来是一些基础设置，英文基本都看得懂吧。接下来输入命令`yo`，然后是一些基础配置，自己配一下就好了。有一点需要提一下，adapter选项可以输入shell，将shell作为adapter是最简单的方式了。

### 运行本地Hubot ###

 接上一步操作，命令行输入`bin\hubot`，以后启动hubot都用这个命令。注意这一步不要进入`bin`文件夹，不然会提示错误。

 	[Fri Oct 19 2018 09:21:47 GMT+0800 (中国标准时间)] INFO hubot-redis-brain: Using default redis on localhost:6379
 	[Fri Oct 19 2018 09:21:47 GMT+0800 (中国标准时间)] ERROR hubot-heroku-keepalive included, but missing HUBOT_HEROKU_KEEPALIVE_URL. `heroku config:set HUBOT_HEROKU_KEEPALIVE_URL=$(heroku apps:info -s | grep web.url | cut -d= -f2)`

 你可能会看到上述的提示信息，这是因为默认情况下，hubot用redis持久化存储，并支持heroku部署，这里我建议删掉这两个配置项，在文件根目录中找到`external-scripts.json`这个文件，把`hubot-heroku-keepalive 和 hubot-redis-brain`这两行删掉，这里可以备份一下这个文件，以便后期还有用到。

 启动之后可以测试你的机器人，输入命令`help`，可以看到打印出了一些信息，输入机器人名+help，可以看到更详细的帮助信息。

 至此一个默认的hubot机器人就部署在你的电脑上了。Linux下操作可能有所不同，不过基本思路差不多。

### weixin Adapter ###

#### 安装hubot-weixin ####

 这一段往后的内容，默认至少了解了一些hubot基础知识，包括adapter，执行自己的脚本等。

 GitHub上有大佬已经写出了基于网页端微信的http的adapter了，链接：[https://github.com/KasperDeng/Hubot-WeChat](https://github.com/KasperDeng/Hubot-WeChat "微信Hubot")，可以说已经写的挺完善的了，不过这个项目也很久没有人维护了，希望能有大佬接手继续贡献新的api什么的。

 按照README.md里步骤一步步执行下去就可以了。接下来写一些需要提的注意点和一些配置安装方面的坑。

 原作者已经将项目传到npm上了，所以可以使用npm直接安装。在配置好默认hubot的文件根目录中，打开`package.json`，在`dependencies`对象的第一行加上`"hubot-weixin": "^1.0.8",`，注意最后的逗号，因为这是json文件。后面的数字是版本号，因为很久没人维护了所以还停留在1.0.8版本。

 命令行中转到项目的根目录下，运行`npm install`，会自动的把`hubot-weixin`下载下来放到文件中。

#### 配置config.yaml ####

 可以直接参考readme，如果遇到问题再往下看吧。

 访问微信网页版[https://wx.qq.com/](https://wx.qq.com/ "微信网页版")，按F12进入前端调试，不同浏览器内核不同调试界面也不同，切换到network选项卡，然后扫码登录，注意这时候能在network里看到很多请求，搜索`webwxinit`找到初始化请求，在请求头部`Request Header`中找到`cookie`，复制到`config.yaml`的`cookie`中，在`Request Payload`（可能不叫这个名字，浏览器不同）的`BaseRequest`中找到4个属性，`Uin, Sid, Skey, DeviceID`，我的`Skey`是空的，是正常的，同样复制到文件里。注意观察一下`RequestURL`是`wx还是wx2`，需要的要替换掉，作者在文档里也写了备注。

 接下来启动吧，`bin\hubot -a weixin`，可能会出现各种错误。详细讲一下可能遇到的报错。

 - `Error: Failed in WxBot getInit`，` ERROR status: 200`这类的错误，一般来说无法初始化都是上一步填那4个属性和`baseUrl, baseUploadUrl`写的有问题。重新登录一次网页微信再试试，每次都会有不一样的4个属性，看着改吧。
 - 提示这俩信息：
 这是前面运行那步没有把`external-scripts.json`里的两项删掉导致的，回前面看看。

 	[Fri Oct 19 2018 13:33:25 GMT+0800 (中国标准时间)] INFO hubot-redis-brain: Using default redis on localhost:6379
 	[Fri Oct 19 2018 13:33:25 GMT+0800 (中国标准时间)] ERROR hubot-heroku-keepalive included, but missing HUBOT_HEROKU_KEEPALIVE_URL. heroku config:set HUBOT_HEROKU_KEEPALIVE_URL=$(heroku apps:info -s | grep web.url | cut -d= -f2)

 如果上面的步骤完成了还是不能启动，建议按照状态码和错误提示一点点debug吧。

 最后讲一点，用这个adapter的时候最好把debug模式打开，不然你也不知道到底启动了没，文件根目录下找到`node_module -> hubot-weixin -> src -> wxLog.coffee`，找到`log = new Log ...`这一行，把`info`改成`debug`，就能看到更多的信息了。在这个模式下启动会看到你的微信好友列表，群成员等等一堆信息，还会看到一直在发送的同步信号。

 博客虽然写到最后了，但是weixin with hubot才刚刚开始，接下来还有很多的扩展。我也在学习开发更多有趣的功能。