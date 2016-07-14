# Respond.js
Respond.js让不支持css3 Media Query的浏览器包括IE6-IE8等其他浏览器支持媒体查询。

Respond.js 是一个快速、轻量的 polyfill，用于为 IE6-8 以及其它不支持 CSS3 Media Queries 的浏览器提供媒体查询的 `min-width` 和 `max-width`特性，实现响应式网页设计（Responsive Web Design）。

响应式布局，理想状态是，对PC/移动各种终端进行响应。媒体查询的支持程度是IE9+以及其他现代的浏览器，但是IE8在市场当中仍然占据了比较大量的市场份额，使我们不得不进行IE低端浏览器的考虑。那么如何在IE6~8浏览器中兼容响应式布局呢？
这里我们需要借助这样一个文件：respond.js。

文件下载地址：[https://github.com/Neveryu/Respond.js/blob/master/respond.js](https://github.com/Neveryu/Respond.js/blob/master/respond.js).

自己在阅读了官方文档之后，进行了一系列测试。友情提示各位朋友，关于respond.js的使用，有一些需要注意的地方，一旦不注意，在IE6-8中就无法显示出来。



###插件原理
既然要实现响应式网页，那么就需要用到媒体查询，媒体查询的核心是`min-width` 和 `max-width`,而IE8以下以及一些其它的浏览器不支持这两个属性，respond.js是怎么做的呢？

* 将head中所有外部引入的CSS文件路径取出来存储到一个数组当中；
* 遍历数组，并一个个发送AJAX请求；
* AJAX回调后，分析response中的media query的min-width和max-width语法（注意，仅仅支持min-width和max-width），分析出viewport变化区间对应相应的css块；
* 页面初始化时和window.resize时，根据当前viewport使用相应的css块。

###使用方法
考虑到IE9是支持CSS3的，所以直接在HTML页面的head标签中添加脚本引入即可：

    <head>
        <link rel="stylesheet" href="style.css">
        <!--[if lt IE 9]>
          <script src="js/respond.js"></script>
        <![endif]—>
    </head>
<span style="color: red;">讲道理，我们是应该将js文件放在html文件的最后，但是repond.js文件，我还是建议你将它放在`head`中，并且放在css文件的后面。</span>
越早引入越好，在IE下面看到页面闪屏的概率就越低，因为最初css会先渲染出来，如果respond.js加载得很后面，这时重新根据media query解析出来的css会再改变一次页面的布局等，所以看起来有闪屏的现象


###核心结论
那么此时，就可以根据基本的实现思路，得到一些书写代码时要注意的地方：
* 需要启动本地服务器（localhost），不能使用普通本地的url地址（file://开头）；
* 需要外部引入CSS文件，将CSS样式书写在style中是无效的；
* 由于respond插件是查找CSS文件，再进行处理，所以respond文件一定要放置在CSS文件的后面
* 另外，虽然把respond放置在head里还是在body后面都能够实现，但是建议放置在head中（具体原因在下面的文档提示中有提到）
* 最好不要为CSS设置utf-8的编码，使用默认（原因详见下面的文档提示部分）



###文档提示
在官方文档当中的一些提示：
* 越早的引入respond.js文件，也就越可能避免IE下出现的闪屏。
* 不支持嵌套的媒体查询
* utf-8的字符编码对respond.js文件的运行有影响
官方API原文：if CSS files are encoded in UTF-8 with Byte-Order-Mark, they will not work with Respond.js in IE7 or IE8.
基本含义就是：utf-8格式的CSS文件字符编码会对插件造成影响。
但是在我使用IE6-8进行测试的时候，都能够正常显示（无论是在css文件中增加charset设置还是在link标签中增加charset设置）。因此，并不是太清楚这个位置bug的含义。
* 跨域可能会出现闪屏（还没有测试，具体情况不详）

###NOTE
* #####Respond.js 和 跨域（cross-domain） CSS 的问题
    如果Respond.js和CSS文件被放在不同的域名或子域名下面（例如，使用CDN）时需要一些额外的设置。请参考Respond.js文档获取详细信息。
* #####Internet Explorer 8 与 box-sizing
    IE8不能完全支持box-sizing: border-box;与min-width、max-width、min-height或max-height一同使用。由于此原因，从Bootstrap v3.0.1版本开始，我们不再为.container使用max-width。
* #####IE兼容模式
    Bootstrap不支持IE的兼容模式。为了让IE浏览器运行最新的渲染模式，建议将此 标签加入到你的页面中：
            `<meta http-equiv="X-UA-Compatible" content="IE=Edge">`
此标签被加入到所有文档页面和案例页面中，以确保在每个被支持的IE浏览器中保持最好的页面展现效果。

* #####Respond.js和@import
Respond.js不支持通过@import引入的CSS文件。例如，Drupal一般被配置为通过@import引入CSS文件，Respond.js对其将无法起到作用。


###Tips
从respond.js的工作原理可以知道，它将head中所有外部引入的CSS文件路径取出来存储到一个数组当中；然后遍历数组，并一个个发送AJAX请求；可以看出这里必须依赖ajax请求css路径才能得到css文件中的mediaquery的内容，那ajax的跨域问题就要解决了；

由于我们的静态资源都是要放在cdn的，responds.js也给出了跨域方法，即引入代理页面。
    //把cross-domain/respond-proxy.html 放到cdn上
    //把cross-domain/respond.proxy.gif 放到当前域服务器上
    <!-- Respond.js proxy on external server -->
    <link href="http://externalcdn.com/respond-proxy.html" id="respond-proxy" rel="respond-proxy" />

    <!-- Respond.js redirect location on local server -->
    <link href="/path/to/respond.proxy.gif" id="respond-redirect" rel="respond-redirect" />

    <!-- Respond.js proxy script on local server -->
    <script src="/path/to/respond.proxy.js"></script>


###Tips
<b>always link stylesheets or write inline CSS before js scripts.</b>
