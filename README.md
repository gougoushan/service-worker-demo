# 用人话来讲解一下Service Worker 和 PWA
用人话来讲解一下Service Worker 和 PWA
标题上为啥要把PWA放在Service Worker后面？因为要理解和开发PWA，Service Worker是决定性因素，为啥这么说，往下看便知晓。

不想因为用户体验而丢失用户，你必须会用Service Worker！--- 致前端开发的小伙伴们

先说一下PWA是个什么鬼，它身份证上的名字是Progressive Web App，看名字就瞬间秒懂了，它就是个网页，只不过做了一些改进，试图抢占APP的市场份额。

针对我们日常使用的APP（美团、饿了么等）来说，除了消息推送以外，APP上的所有功能在网页上都能实现。但是你可能忽略了很关键的一点，就是在没有网的情况下，APP还能正常打开，但是网页呢，你只能打开一个提示无法连接的页面，可你要说了，APP即便能打开又怎么样，没网还是用不了啊，但你别忘了，有些APP运行是不需要网的，比如计算器，没网APP照样用，可如果做个网页版的计算器，没网可就完犊子了！所以，网页和APP能否一较高下，需要解决的关键问题就是“没有网你咋办？”，Service Worker的出现就是来解决这个问题的，也正是因为有了Service Worker，PWA才敢和APP叫板。

简单介绍一下Service Worker，它实际上就是浏览器提供的一组JavaScript API，这组API的功能肯定是相当丰富了，但是咱得捞干的说，对咱们最重要的功能就两个，一个是对网页及其资源的离线缓存，另一个是对浏览器请求的拦截。

说到这你可能明白了，要实现网页在没有网的情况下还能用，对网页做离线缓存就可以了，HTML5早就有这个功能，你说的应该是AppCache吧，这货虽然可以达到缓存的目的，但是本质上来讲，它和你在浏览器上选择菜单里的“网页另存为”，把网页保存到本地，没啥太大区别。这就意味着，如果你想清除缓存，sorry，作为程序员的你，办不到，办不到，办不到！你还得劳驾用户自己通过操作浏览器的方式清除缓存。要知道，人家APP对于缓存的操作可是想咋整就咋整，网页想跟APP怼，就凭AppCache这么个玩意，肯定拿不出手啊！所以下面咱们就来看看Service Worker是怎么做离线缓存的。

我们知道JavaScript是单线程运行的，可是浏览器这次史无前例的为了Service Worker单独开了一个线程，也就是说当浏览器加载你的网页的时候，网页上的脚本运行在一个线程中，而Service Worker自己运行在另一个线程中，他们互不干涉内政，你不用担心Service Worker会影响你原本的网页加载效果，所以，你应该能够想到，我们必须把Service Worker的代码单独放在一个js文件中，我们通常会建立一个service-worker.js文件，那么这个文件是怎么和你的网页搞到一起的呢，注意，现在需要在你的网页中写代码了

```JavaScript
//这里是要检查用户的浏览器是否支持Service Worker
//从这里就可以看出Service Worker就是navigator的一个属性对象
if ('serviceWorker' in navigator) {
  //添加一个网页载入事件的监听器
  window.addEventListener('load', function () {
    //当网页载入完成后，把service-worker.js文件注册到网页中
    navigator.serviceWorker.register('/service-worker.js')
  });
}
```

上面代码执行完以后，Service Worker就和你的网页息息相关了，他会一直为你的网页服务，直到它被卸载为止，其实代码还是非常简单的，但是说了这么多，代码也写了，还是没说Service Worker是怎么做缓存的啊，你要注意，上面代码是在你的网页中的，并不是在service-worker.js文件里的，这个代码的作用只是把service-worker.js注册到你的网页上，让他起作用，所以，真正实现缓存功能的代码肯定在service-worker.js文件里啦！那么这里的代码应该怎么写呢？

可以想象这里的代码一定很多很复杂，但是我要恭喜你的是，这里的代码根本就，不用写，不用写，不用写！因为我们有工具来生成它，他就是，当当当当，sw-precache，它身份证上的名字是Service Worker Precache，其实上面写在你网页里的代码也是可以自动生成的，但是需要你会用webpack，考虑到你可能还不了解webpack，我们就不提这闹心的事了，幸好sw-precache还有命令行下的使用方式。

sw-precache这货实际上是个nodejs模块，所以要使用它你还得先把nodejs安装好，至于怎么安装nodejs，等nodejs社区倒闭了我再告诉你。下面说怎么安装sw-precache，在命令行下运行如下命令：

`
npm install --global sw-precache  #（别忘了回车啊）
`

根据我的经验，你的安装过程中不会出现任何问题，简单的说一下--global这个参数是啥意思，它是告诉nodejs这个模块要进行全局安装，这就意味着你在系统的任何目录下都可以执行sw-precache的命令了，所以，现在你就可以在你的web服务器上创建一个网站，把我提供给你的 [demo](https://github.com/gougoushan/service-worker-demo/archive/master.zip) 文件放在网站目录下，打开命令行cd到网站目录，执行下面命令：

`
sw-precache  #（我再告诉你，别忘了回车啊）
`

这时在当前目录会生成一个service-worker.js文件，没错，这就是我们想要的，一切都搞定了！现在打开浏览器（别说我没提醒你用Chrome，用别的浏览器行不行以后再说），访问你的这个网站，然后你肯定迫不及待的要测试一下没网它还能不能正常打开，OK，但是你可别去断开网络连接啊（我估计你不会傻到这份上），因为你访问的是本机地址（例如：localhost），没网也照样能访问，所以你应该做的是关闭你的web服务器，然后你再去刷新Chrome（狂按F5吧），怎么样？

咱们说了，Service Worker是在独立线程中运行的，那是不是可以观测一下它运行的咋样啊？必须的！打开下面这个URL

chrome://inspect/#service-workers

这时你可以看到当前正在运行的Service Worker线程，如果你先看的更详细一点，就打开这个URL

chrome://serviceworker-internals

这时连没运行的Service Worker线程也能看到了，并且还有各种属性，而且如果你看哪个Service Worker线程不顺眼还可以把它卸载了（Unregister）。

你可能觉得这似乎有点太简单了，这样就实现了离线缓存吗？是的，当执行sw-precache命令的时候，它默认把当前目录下的所有文件都进行缓存，并且加上缓存逻辑形成了一个service-worker.js文件。但有时候我们并不想缓存所有文件，只想缓存指定目录下的文件，那你可以给sw-precache提供一个配置文件来实现：

```JavaScript
// sw-precache-config.js
module.exports = {
  staticFileGlobs: [ //指定要进行静态缓存的文件
    'app/css/**.css', //缓存所有css文件，以下的配置语法都类似
    'app/**.html',
    'app/images/**.*',
    'app/js/**.js'
  ],
  runtimeCaching: [{
    urlPattern: /index\\.php/,
  }]
}
```

然后执行下面的命令：

`sw-precache --config=sw-precache-config.js --verbose`

上面的配置文件有一个配置节点你一定会感兴趣，runtimeCaching，这个节点是用来配置动态缓存的，静态缓存和动态缓存的区别是：静态缓存意思就是在网页加载的时候就明确的告诉浏览器需要缓存什么文件，而动态缓存的意思是，在网页加载之前我们还不能确定哪些文件需要缓存，我们要在网页的运行期通过程序来判断哪些要缓存哪些不缓存。开头咱们说了，原生APP对于缓存可是想咋整就咋整，现在有了动态缓存，咱网页APP就真的能和原生APP磕了，还记得咱们说Service Worker有两个对咱们最重要的功能吗？一个是离线缓存，一个是请求拦截，这个动态缓存就是通过请求拦截来实现的。

以上这些都只是雕虫小技，如果Service Worker只能干这些事就太名不符实了，它还有一个功能说出来吓死你，它能给用户推送消息通知，不信吗？你用没用过网页版的微信啊？如果你是第一次打开网页版微信，会出现一个提示框，问是否允许微信给你发送通知，如果你点击允许了，以后微信有消息，就会在右下角弹出消息通知。呵呵，老铁们，有了这个功能，咱WebAPP和原生APP还差啥？

说到这吧，下次咱们具体来说说动态缓存！
