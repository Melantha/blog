翻译自：[debugging-react-performance-with-react-16-and-chrome-devtools](https://building.calibreapp.com/debugging-react-performance-with-react-16-and-chrome-devtools-c90698a522ad)

react是现在前端主要框架之一，不仅因为它出名，更因为它出色的渲染性能。react的虚拟DOM以高效的渲染性能闻名，但是在我们项目会出现根本感受不到react高效性能的情况，这是我们该怎么定位，怎么修复它呢？


今天我将用真实的react 代码和chrome工具来证实chrome新的强大的性能追踪工具的特性以及怎么用它来诊断出渲染缓慢的组件。

当然新的性能工具需要满足下面3个条件：
1. 在开发模式中用React 16
2. 用source-maps导出你的js（这个是可选的，但是强烈推荐你）
3. 使用Chrome或者Chromium’s Dev Tools

### 在Chrome中设置你的audit
首先，你需要足够的空间，最好把开发者工具弹出来，使它和你的屏幕一样大，那样是最好的。

[![1_-vtAWWryE6dtsPXkuOU7xQ.gif](https://user-gold-cdn.xitu.io/2019/3/14/1697b6a8e2ca425c?w=800&h=261&f=gif&s=331014)](https://user-gold-cdn.xitu.io/2019/3/14/1697b6a8e2ca425c?w=800&h=261&f=gif&s=331014)

我们应该用chrome的audit 功能模拟真实环境中代码的运行情况，这是必不可少的。不是每个人都有价值3000美元的电脑或者最新的手机。

幸运的是，Chrome对于设备模拟覆盖广泛，我们能够降低javascript的执行速度，通过这个我们能清晰的发现性能问题，还有额外的好处，如果你能做到在硬件一般的设备运行快速运行，那么桌面上就能体验飞一般的感觉。
[![1_aL8cq4bJxsH4RcTFblGfBg.gif](https://user-gold-cdn.xitu.io/2019/3/14/1697c2943d722241?w=800&h=188&f=gif&s=455785)](https://user-gold-cdn.xitu.io/2019/3/14/1697c2943d722241?w=800&h=188&f=gif&s=455785)
降低你得设备至少到4x就将和Motorola Moto G4（摩托罗拉手机）一样的性能

### 查看性能跟踪
在开发者模式，以及react 16版本渲染的条件下，它会为每一个组件创建标记与相关具体的调用。

首先，打开（本地的）需要测试的页面，点击‘Start profiling and reload page’按钮，这样就创建了一个对页面的性能分析记录（a performance profiler audit）。

这将会为当前页面记录性能轨迹。在页面加载完成几秒钟之后，Chrome将会自动停止记录。
[![1_KXQKkbo8Ujfd8JcRDK9RAA.gif](https://user-gold-cdn.xitu.io/2019/3/15/1697f42f5ba6433b?w=800&h=465&f=gif&s=707818)](https://user-gold-cdn.xitu.io/2019/3/15/1697f42f5ba6433b?w=800&h=465&f=gif&s=707818)
或者，使用键盘快捷键：⌘+⇧E

一旦你开始记录，你的窗口看起来是这个样子：

[![1_JH7OdBXRtDjxLUqGT3b51A.png](https://user-gold-cdn.xitu.io/2019/3/15/1697f47fa81fe008?w=1200&h=940&f=png&s=204930)](https://user-gold-cdn.xitu.io/2019/3/15/1697f47fa81fe008?w=1200&h=940&f=png&s=204930)
我们能看到一条从左到右的记录我们页面加载和初始化的时间轴

我想花一两秒钟指出一些对性能工具使用的新人来说不太明显的事情。
1. 如上图中①所示，红色的指示横条说明在时间轴这部分占用了大量cpu资源（较长的任务执行），我们应该对这个地方进行分析
2. performance顶部窗口图表使用不同的颜色代表不同类型的活动。每个类别都需要不同的分析，修复，造成性能的原因也不同。


现在我们先聚焦在“Scripting”（JavaScript运行时性能）

![JavaScript运行时性能](https://user-gold-cdn.xitu.io/2019/3/15/169802f95c22c484?w=800&h=727&f=gif&s=4081149)
展开窗口，单击打开用户计时，用截屏时间线来分析你的页面如何绘制的👀

接下来我们要去分析那些cpu占用高显示红色的部分。我们可以看到在整个性能跟踪期间页面渲染的元素。

![](https://user-gold-cdn.xitu.io/2019/3/15/169803452d317532)
在图表区域拖拽可以缩放时间轴，你注意到emoji ⚛️了吗?

一缩放立即就会显示用户花费时间信息，然后我们观察到一个叫Pulse的组件，它花费了500ms时间渲染。

在Pulse组件的下面，可以看到它的子组件的渲染，尽管组件的大小表明它们渲染不会花费这么长时间。

### 发现执行慢的函数

![](https://user-gold-cdn.xitu.io/2019/3/15/16980436151dba97?w=1200&h=506&f=png&s=133963)
1. 单击你要分析的组件，在这里就是Pulse。然后我们只需关注在Pulse组件执行缓慢的部分。
2. 选择性能开发工具中“Bottom-Up”.
3. 按照总时间（total time）从高到低排序，当然，你也可以按照自己本身时间（Self time）或者除了URL以外东西排序。总之，怎么适合你分析，怎么排序最好。
4. 在你需要分析代码的地方有个箭头，点击它，在这里就是map 看起来是可疑的。它被调用了很多次，总共执行时间为90ms。
5. **这就是你为什么需要sourcemaps的原因：点击 `MetricStore.js:59`将会打开代码并定位到59行**


![](https://user-gold-cdn.xitu.io/2019/3/15/169804fcf8e34433?w=800&h=986&f=png&s=261675)
在代码执行花费较长的左边就会有一个时间标识（大概这不是我向世界展示我代码的最佳时机）

### 当你知道怎么原因改进它是简单的
在过去的几周里，我通过这样的方法实现了之前很复杂，需要花费很多解决的代码的优化工作。现在我完全知道如何以及在哪里寻找JavaScript性能问题。我也确信在未来我能写出更好的代码。

...
广告滤过
...

### 为什么React的分析工具会在Chrome里面？
在react内部，react发布的这些工具也是通过[User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API)，它就会在你的代码里面添加时间标记，而Chrome利用User Timing API实现这个功能。

> 想象一下一个组件初始化花费0.4s，用户点击购买按钮花费15s时间

关于User Timing API的介绍，可以看看[ Alex Danilo](https://twitter.com/alexanderdanilo)在2014年发表的[User Timing API](https://www.html5rocks.com/en/tutorials/webperformance/usertiming/)，这是名副其实的HTML5 基石的文章。

所有的主流浏览器都已经支持User Timing API，但是在Chrome中通过debugging更简单方便调试一个React应用还需要走一段时间的路。


### 最优化你的JavaScript
我衷心的希望这篇文章能够提升你性能分析的水平。请留下评论，分享你学到的东西或者改进你的应用的方法🙋！


如果有错误或者不严谨的地方，请务必给予指正，十分感谢!
### 参考
1. [https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool?hl=zh-cn](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool?hl=zh-cn)