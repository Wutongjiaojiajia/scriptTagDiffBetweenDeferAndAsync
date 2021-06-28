# 图解script标签中的async与defer属性

## 前言

`script`标签用于加载脚本与执行脚本，在前端开发中可以说是非常重要的标签了。直接使用`script`脚本的话，`html`会按照顺序来加载并执行脚本，在脚本加载&执行的过程中，会阻塞后续的`DOM`渲染。

现在大家习惯于在页面中引用各种第三方脚本，如果第三方服务商出现了一些小问题，比如延迟之类的，就会使得页面白屏，针对上述情况，`script`标签提供了两种方式来解决问题，就是加入属性`async`以及`defer`，这两个属性使得`script`标签加载都不会阻塞`DOM`的渲染。

而存在以上两个属性，那就说明这两个属性肯定存在差异的。

## defer

如果`script`标签设置了defer属性，则浏览器会异步下载该文件并且不会影响后续`DOM`的渲染。如果有多个设置了`defer`属性的`script`标签存在，则会按照顺序执行所有的`script`，`defer`脚本会在文档渲染完毕后，`DOMContentLoaded`事件调用前执行。

针对上述理论，我们编写了一段测试案例，页面包含了两个`script`标签的加载，给他们都加上了`defer`标识，其中给`script1.js`添加了1s的延迟，给`script2.js`添加了2s的延迟。

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628152514874.png" alt="image-20210628152514874" style="zoom:50%;" />

下图是页面加载的过程和`script`脚本的输出顺序。不难看出虽然`script1.js`的加载时长比`script2.js`短，但是因为defer的限制，所以`script1.js`只能等前边的脚本执行完毕后才能执行。

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628153819618-4865921.png" alt="image-20210628153819618" style="zoom: 33%;" />

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628155212350-4866742.png" alt="image-20210628155212350" style="zoom:33%;" />



## async

`async`属性会使得`script`脚本异步的加载并在允许的情况下执行，而`async`的执行并不会按照`script`标签在页面中的顺序来执行，而是谁先加载完谁先执行。

修改上述案例代码如下：

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628160022121.png" alt="image-20210628160022121" style="zoom: 50%;" />

下图是页面加载的过程和`script`脚本的输出顺序。虽然页面加载时长上并没有什么变化，但是可以注意到一个细节的是，`DOMContentLoaded`事件触发并不受`async`脚本加载的影响，在脚本加载前就已经触发了`DOMContentLoaded`。

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628160515332.png" alt="image-20210628160515332" style="zoom:33%;" />

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628160627492.png" alt="image-20210628160627492" style="zoom:33%;" />

## 总结

下面来画多个脚本加载时的甘特图，拿四个不同的颜色来标明各自代表的含义：

<img src="/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628165937483.png" alt="image-20210628165937483" style="zoom:50%;" />

### 普通script

文档解析过程中如果遇到`script`脚本，就会停止页面的解析进行下载，而Chrome会进行一个优化，如果遇到`script`脚本后会快速查看后面有无需要下载的其他资源，如果有的话，会先下载那些资源，然后再进行下载`script`所对应的资源，这样能够节省一部分下载的时间。

资源的下载是在解析过程中进行的，虽然说`script1.js`会比较快加载完毕，但是他前面的`script2.js`并没有加载并且执行，所以他只能处于一个挂起的状态，等待`script2.js`执行完毕后再执行。当这两个脚本都执行完毕了再继续解析页面。

![image-20210628172149936](/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628172149936.png)

### defer

文档解析时遇到设置了`defer`属性的脚本，就会在后台进行下载，但是并不会阻止文档的渲染，当页面解析完毕后，会等到所有的`defer`脚本加载完毕并按照顺序执行，脚本执行完毕后会触发`DOMContentLoaded`事件。

如果你的脚本代码依赖于页面中的`DOM`元素（文档是否解析完毕），或者被其他脚本文件依赖，则使用`defer`属性。

![image-20210628172241073](/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628172241073.png)

### async

`async`脚本会在加载完毕后执行，而async脚本的加载不计入`DOMContentLoaded`事件统计，下图的两种情况都是有可能发生的。

如果你的脚本并不关心页面中的`DOM`元素（文档是否解析完毕），并且也不会产生其他脚本需要的数据，则使用`async`属性。

![image-20210628172423241](/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628172423241.png)

![image-20210628172641002](/Users/huangjiahao/personalData/技术笔记文章/图解script标签中的async与defer属性/image-20210628172641002.png)