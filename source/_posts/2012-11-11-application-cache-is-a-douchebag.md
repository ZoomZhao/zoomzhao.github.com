---
layout: post
category : front-end
title: Application Cache 就是个坑
date: 2012-11-11 22:29:33
tags : [Application Cache, AppCache, 坑]
---

- by  [Jake Archibald](http://www.alistapart.com/authors/a/Jake%20Archibald "Jake Archibald")  
- 原文地址 [Application Cache is a Douchebag](http://www.alistapart.com/articles/application-cache-is-a-douchebag/ "Application Cache is a Douchebag")

![application-cache-is-a-douchebag](http://www.alistapart.com/d/application-cache-is-a-douchebag/application-cache-is-a-douchebag.jpg)

早上好，我们最近发布了 [我们的 Mobile 网站](http://m.lanyrd.com/)，其中使用了离线缓存技术。我将离线缓存部分的代码简化成了[一个小的实例](http://appcache-demo.s3-website-us-east-1.amazonaws.com/localstorage-cache/)，并把代码放到了 [Github](https://github.com/jakearchibald/appcache-demo) 上。不过在深入代码之前，请允许我向您讲述一个真实的故事。  

<!-- more -->

故事发生在一个大家彼此很陌生的派对上，我躲在一群尝试介绍自己的人群之中。这时候，一个漂亮的女士走过来对着一个略显羞涩的人说，"我是 Dev，你叫什么名字？"  

"呃，我是 LocalStorage"，他回答的很不自然，"我提供了一个脚本接口来管理跨页面和会话的文本存储"。  

"他其实只是个简单的柜子"，另外一个人打断道，其余人也纷纷嘲笑 LocalStorage 对于资源的浪费。我跟他很熟，所以我并没有参与其中。  

这个时候，人群中的另一个人尖着嗓子说，"嗨，我是 ApplicationCache"，说话的同时，他伸手去与 Dev 握手，"我仅仅通过增加一个文件成功的提升了狗屎般的离线体验，小而美，不需要脚本控制"，他在说脚本控制的同时做了个砍价的动作。与此同时，我却是咬牙切齿，因为我知道他过分的夸大了他的能力，坏消息是其余的人并没有意识到这一点。但是如果在这种情况下指出他在胡说八道，我肯定会被认为是个混蛋。  

没能在那个晚上指出 ApplicationCache 的问题，让我感到非常不舒服。看到那些只是顺路看了下 ApplicationCache 的人夸奖他的易用性真是让人痛苦。我想我必须来说点什么：我在这里告诉你 ApplicationCache 就是个坑。  

并不是说 ApplicationCache 没有用，或者是需要尽量避免使用，只是在什么情况下使用以及怎么使用的问题上需要非常的谨慎。一旦出了差错，混乱会被传递给用户。通过了解我们使用 ApplicationCache 的痛苦经历，你也许可以了解到能够从 ApplicationCache 中得到什么以及应该怎么使用它。  
  
### 离线访问的适用场景是怎么样的  
我们在接入网络方面变得越来越好了，可以却依然不能任何什么时候都保持在线。比如说正在一辆在萨塞克斯西部平原上穿行的火车上写这篇文章的我。另外，当我在国外使用数据服务的时候我几乎可以听到网络提供商办公室里开香槟庆祝的声音。在数据漫游时使用网络会让我大放血，但网络中有我需要的数据，有些时候又不得不通过网络连接来获得它。  

可以在离线情况下访问的网站大概可以分为两类，一类"可以离线使用"的网站，而另一类则"可以离线查看"的网站。  

"可以离线查看"的网站包括 [Wikipedia](http://www.wikipedia.com/), [YouTube](http://youtube.com/), 和 [Twitter](http://twitter.com/)。他们复杂的处理都在服务器上进行，虽然可以获得很多的数据，但是在用户只能使用其中很小的一部分。  

"可以离线使用"的网站有 [Cut the Rope](http://www.cuttherope.ie/), [CSS Lint](http://csslint.net/), 和 [Google Docs](http://docs.google.com/)。这些网站复杂的业务逻辑很多是在客户端实现的。他们只提供了一定量的数据，但是可以通过各种方法是用这些数据，用户甚至创造自己的数据。这正是 ApplicationCache 被设计时想要针对的情况，我们来首先来看一个关于它的简单介绍。  

### "可以离线使用"的网站  
[Sprite Cow](http://www.spritecow.com/) 是一个可以离线使用的网站，CSS sprites 对于[提高性能](http://css-tricks.com/css-sprites/)是很有好处的，但是在 sprite 页中找到一个元素的大小和位置是很复杂的。Sprite Cow 加载 sprite 页面，然后产生 CSS 来展示其中特定的部分。它包括一个 HTML 文件，一些静态文件，所有的处理都会在客户端执行，服务器只负责提供静态文件。  

在火车上能像使用 Native App 一样使用 Sprite Cow 真的是很好的体验。为了做到这样，我们创建了一个 manifest 文件，其中列出了网站需要的所有静态文件：
 
    CACHE MANIFEST
    assets/6/script/mainmin.js
    assets/6/style/mainmin.css
    assets/6/style/fonts/pro.ttf
    assets/6/style/imgs/sprites1.png
    ...

然后将 manifest 文件通过一个属性链接到 html 页面上。  

    `<html manifest="offline.appcache">`

HTML 页面本身没有在 manifest 文件中列出，但是包含 manifest 的页面自身会成为 manifest 的一部分。  

实践表明，如果你在线的情况下访问了 [Sprite Cow](http://www.spritecow.com/)，你可以在以后离线时访问他。  

Chrome 的 Web Inspector 中的 resource 标签可以展示 manifest 文件中记录的文件，它属于属于哪个页面。如果你需要清除这些缓存，访问 [chrome://appcache-internals/](chrome://appcache-internals/) 就可以了。  

首先，这看上去像是一个很神奇的解决方案，不幸的是，我在说这会是个简单介绍的时候撒谎了，ApplicationCache 的规则更像是一个洋葱，它有很多很多层，随着你慢慢剥开它，眼泪也会慢慢流下来。  

## 坑#1：即便在线，文件也从 ApplicationCache 中加载
当你访问 Sprite Cow 的时候，你会立即获得缓存中的版本，当页面渲染结束后（实际上不一定在渲染结束的情况下进行），浏览器会查看 manifest 文件以及被缓存的文件的更新情况。  

这看上去像是很奇怪的处理方式，这决定了浏览器不用等到连接 Timeout 来决定你处于离线状态。  

ApplicationCache 会抛出 `updateready ` 事件来告诉我们内容有更新，但是我们并不能在这个时候简单的刷新页面，因为用户可能正在使用他们已经获得的旧版本。  

这并不是一个大问题，因为旧版本可能也是足够好的，如果需要的话我们可以显示"发现了一个更新，请刷新页面获取更新"的信息。你可能曾经在 Google 的应用中看到过它，比如 Google Reader 或者是 Gmail。  

还记得我在四段之前说 ApplicationCache 会在页面渲染结束后查看是否有更新吗? 我又撒谎了。

## 坑#2：ApplicationCache 只在 Manifest 文件改变的时候更新  
HTTP 已经有缓存模型了，每个文件可以定义怎样被浏览器缓存。最基本的，每个独立的文件可以声明"永远不要缓存我"或者"与服务器进行验证，它会告诉你有没有更新"或者"直到 2022 年 4 月 1 日我都是可以使用的"。  

但是，如果在你的 manifest 中有 50 个 html 页面，每次你在线的情况下访问他们的时候，浏览器会创建 50 个网络请求来确认他们是不是需要更新。  

作为一个不寻常的解决方式，浏览器只会在 manifest 文件发生更新时查看 manifest 文件中列出的文件是不是有更新。 manifest 文件比较是以二进制方式进行的。  

这个处理对于通过 CDN 分发并且永远不会改变的静态文件来说是透明的。当 CSS/JavaScript 等等改变的时候，它们会使用与之前不同的 url，这意味着 manifest 文件改变了。如果你不熟悉缓存以及 CDN，请先查看 [Yahoo 的性能最佳实践指引](http://developer.yahoo.com/performance/rules.html#expires)。  

有些资源不能简单的改变他们的 url，比如说我们的 HTML 页面。除非更新了 manifest 文件 ApplicationCache 不会查看文件的更新。最简单的方式是，在 manifest 文件中增加注释。

    CACHE MANIFEST  
    # v1
    whatever.html  

在 manifest 文件中，注释以`#`开始，如果我更新了 html 页面，我需要更新注释到 `# v2`，来触发更新。你可以使用脚本通过类似 ETag 的方式修改在 manifest 文件中的注释，这样每个文件的改变都可以保证 manifest 文件能够更新。  

但是，更新 manifest 文件中的文本并不能保证其中的资源会访问服务器获得更新。  

## 坑#3：ApplicationCache 只是个附加的缓存，而不是替代品  
当浏览器更新 ApplicationCache 时会像平常一样请求 manifest 文件的地址。它遵循普通的缓存逻辑，如果一个元素的头部声明"在 2022 年 4 月 1 日之前我都是足够新鲜的"，浏览器会假设其是最新的直到 2022 年 4 月 1 日，这会阻止 manifest 的更新。  

这是件好事，因为你可以通过它在 manifest 文件改变时减少浏览器需要发送的请求数目。  

但这也会坑人，所以有些人尝试服务器不提供缓存头。但是在不提供的情况下，浏览器会猜测文件的缓存过期时间，你可以更新 html 页面以及 manifest 文件，但是浏览器不一定会更新它，因为他"猜测"文件是不需要更新的。  

服务器提供的所有文件都应该包含缓存头，他们对于 manifest 中的文件以及 manifest 自身来说都是很重要的。如果文件会被频繁更新，它应该带上 `no-cache` 头部。如果文件偶尔更新，`must-revalidate` 是个更好的选择，比如 `must-revalidate` 对于 manifest 自身来说是个很好的选择。好吧，有点跑题……  

## 坑#4：永远永远不要长期缓存 Manifest  
你可能觉得 manifest 文件是个静态文件，就像"在 2022 年 4 月 1 日之前我都是足够新鲜的",然后再更新的时候更新 manifest 文件的 url。  

别！千万别这样！  

记住坑#1中提到的内容，当用户第二次访问页面时，他们会获取 ApplicationCache 的版本，如果已经更改了 manifest 文件地址的话，很不幸的，用户缓存的版本依然指向旧的 manifest 文件，它就永远不会更新了，永远。  

## 坑#5：未缓存的资源不会在缓存的页面中展示  
如果你缓存了 index.html，但是没有缓存 cate.jpg，文件即便是在线的情况下也不会在 index.html 中显示，实际上这是预期的行为，[自己看吧](http://appcache-demo.s3-website-us-east-1.amazonaws.com/without-network/)。  

为了禁止这个行为，需要使用 manifest 中的 NETWORK 段
 
    CACHE MANIFEST
    # v1
    index.html
    NETWORK:  
    *

字符 * 表示，浏览器应该允许所有缓存页面中的的非缓存连接。这里，你可以看到[设置 NETWORK 之后的实例](http://appcache-demo.s3-website-us-east-1.amazonaws.com/with-network/)。显然，这些链接在离线情况会失败。  

很好，你已经完成了使用 ApplicationCache 的简单示例。是的，虽然这只是个简单示例。让我们来点厉害的吧。

### "可以离线查看"的网站  

就像我在文章开始提到的那样，Lanyrd 最近发布了[一个 Mobile 网站](http://m.lanyrd.com/)，用户可以查看会议日程，地点，参与者等。离线访问这些数据对旅行和面临数据漫游费用的人来说是非常重要的。  

离线所有内容简直太大了，但是通常一个用户只会对他们参与的事件感兴趣。  

最近的 [Dive into HTML5](http://diveintohtml5.info/) 给我们[一个关于离线 Wikipedia 的示例](http://diveintohtml5.info/offline.html#fallback)，那是另外一个"可以离线查看"的网站。它的工作方式是每个链接使用一个几乎为空的 manifest 文件，当用户在网站中浏览时，这些网页立即成为他们缓存的一部分，当离线时，他们能够访问之前查看过的任意网页。  

Wikipedia 的解决方案非常简单，但是由于规范中存在一些诡异的地方，这又是非常惨的。对于初级用户来说，用户没有收到任何指示来表明哪些内容在离线情况下是可用的，并且没有可以获取这些信息的 JavaScript API。我们可以下载 manifest 文件，然后使用 JavaScript 解析它，但是所有的  Wikipedia 的页面是隐式缓存的，所以他们没有在 manifest 文件中被列出。  

更进一步，坑＃1：缓存中的版本会被显示，而不是服务器端的版本。当用户第一次访问时，页面会被冻结，但是我们在坑＃2中发现，我们可以通过修改 manifest 文件来触发浏览器查看更新。但是我们什么时候改变 manifest 文件呢？每当一个 Wikipedia 的条目被更新？那样就会变的过于频繁，实际上如果 manifest 文件在更新的开始以及结束之间被修改，浏览器会[认为此次更新失败(step24)](http://www.whatwg.org/specs/web-apps/current-work/multipage/offline.html#downloading-or-updating-an-application-cache)。  

更新的频率是个问题，但是更新的大小才是真正的杀手，想想你所看过的 Wikipedia 的数量吧，几百？几千？一个 AppCache 的更新会包括下载所有的页面。AppCache 没有提供删除隐式缓存元素的方法，所以数量会不断的增长，增长，直到达到浏览器的缓存最大限度，然后世界崩溃。这不是什么好事。  

### 我们希望从离线网站获得什么？  
我需要从一个提供离线能力的网站获得的是：  

- 在线的情况下显示最新的数据，和没有 ApplicationCache 一样  
- 允许我们（开发人员）控制哪些内容被缓存，什么时候被缓存以及怎样被缓存  
- 允许我们提供给用户一些控制的能力，有可能以“保存为离线”或者“稍后阅读”按钮  
- 每次对页面的访问必须提供给浏览器供离线显示需要的所有内容  

ApplicationCache，甚至夸大了它的作用的那些说法，都没有让这些变得简单。如果页面中包含了 manifest，它就会被缓存，你没办法用一个页面告诉浏览器所有被缓存的内容。  

### 限制 ApplicationCache 的使用范围  
最简单的让一个页面像没有 ApplicationCache 一样的方法是提供它时不包含 manifest。我们可以通过插入一个不可见的 iframe 指向包含 manifest 的页面来告诉浏览器需要缓存的内容。

让我们转个弯。在线的情况下[访问这个页面](http://appcache-demo.s3-website-us-east-1.amazonaws.com/offline-iframe/)。之后你可以在离线的状态下访问[这个页面](http://appcache-demo.s3-website-us-east-1.amazonaws.com/offline-iframe/offline.html)。第一个页面并不是缓存的一部分，所以用户一直可以通过它获取最新的内容。  

### Fallback  
通过 ApplicationCache 的 manifest 文件可以定义一个如果一个请求失败的时候 Fallback 使用的资源。  

    CACHE MANIFEST
    FALLBACK:  
    / fallback.html  
    /assets/imgs/avatars/ assets/imgs/avatars/default-v1.png  

这个 manifest 文件告诉 ApplicationCache 在任何网络请求失败的情况下显示 fallback.html，除非包含 /assets/imgs/avatars/ 的请求失败，这种情况下一个 Fallback 的图片会被使用。  

了解 Fallback 实际怎么工作，[访问这个页面](http://appcache-demo.s3-website-us-east-1.amazonaws.com/simple-fallback/)。在没有网络请求的情况下再次访问页面，Fallback的页面会被显示。需要注意它强制重定向的方式，原始页面的 url 会被保留，这是很有用的。  

顺便，存在 Fallback 资源[解决了在坑＃5中提到的网络规则中遇到的疑难杂症](http://appcache-demo.s3-website-us-east-1.amazonaws.com/with-fallback/)。现在同一域名中的连接都会被允许，但是对于其他域名来说网络通配符还是需要的。  

现在我们更近了一步，我们在普通页面没有成功加载的情况下显示一个被缓存了的页面。  

### 只对静态内容使用 ApplicationCache  
说“静态内容”，我的意思是内容从来不会改变。可以是图片，脚本，样式表和我们的 Fallback 页面。  

    CACHE MANIFEST
    js/script-v1.js
    css/style-v1.css
    img/logo-v1.png
    FALLBACK:
    / fallback/v1.html
    /imgs/avatars/ imgs/avatars/default-v1.png

如果我们需要改变 JavaScript，我们上传一个新的文件，并且使用新的 url，比如：script-v2.js。  

这样就可以解决坑1＃以及坑2＃带来的问题：用户永远被不会提供过期的脚本或者样式，因为它们在改变的时候会拥有一个新的 url。我们不需要去处理一个包含注释的版本好的 manifest，因为 url 中的文本改变已经足够触发刷新。所有的资源都会拥有一个未来过期时间，这样只有更新了的文件需要 Http 请求来更新内容。  

## 坑6＃：再见吧，条件下载  
等等，你不觉得整篇文章到现在还没有提到任何关于响应式设计的部分吗？  

你有两套设计图片吗？是不是其中一个更小，以便使用 Mobile 设备的用户查看网站？你使用 Media Queries 来决定显示哪一个图片吗？好吧，ApplicationCache 讨厌你，讨厌你全家。  

所有需要在网站中渲染的图片都需要在 manifest 中，并且浏览器会将所有图片下载。在这种响应式图片的情况下，用户最终会下载同样资源的所有版本，这显然违背了我们的意愿。用桌面版的图片吧，在客户端通过 [CSS background size](https://developer.mozilla.org/en/CSS/background-size) 重新处理图片的大小。  

如果 Mobile 版本有一个完全不同的设计，至少将他们同高分辨率图片一起放置到一个 sprite 中，这样他们可以同时从 png 压缩中获益。  

对于字体来说也是同样的道理。我看到过一些人建议使用[很多的字体格式](http://speakerdeck.com/u/jaffathecake/p/in-your-font-face?slide=38)，这对于普通网站来说是不错的，但是我们不能把他们都放到 manifest 文件里。对于离线用户，只使用 Trye Type Fonts (TTF) 吧，“嗨，未来不是属于 Web Open Font Format (WOFF) 的吗？”，是的，有可能，但是只是因为法律的原因。WOFF 相对于 TTF 并没有什么优势可言。是的，WOFF 可以在创建时进行压缩，但是并没有比 Gzip 的 TTF 好。另外，[WOFF 不支持很多的旧版本浏览器](http://caniuse.com/woff)，而 TTF 支持的浏览器多很多。  

随他去吧，回到 Application Cache。
### 对于需要离线的动态内容使用 LocalStorage  
我们不能离线下载所有内容，它们确实太多了。我们希望用户获得他们离线情况下希望获得的内容。我们可以使用 LocalStorage 来保存数据。  

是的，LocalStorage 就是个柜子，但是它是个使用很简单的有用柜子。在同一域名下的页面内，你可以放置任何的 text 数据到 LocalStorage 然后在之后获取它。  

LocalStorage 是存储在硬盘的，虽然耗费不大，但是也[不是完全没有耗费](http://，我们应该控制calendar.perfplanet.com/2011/localstorage-read-performance/)的。所以我们应该控制读写的数量，只在必须的时候读写它。  

我们给每一个存储的页面存储一个列表，然后另外设置一个列表跟踪我们离线缓存的内容和页面的标题，这意味着通过一次读取我们就可以列出我们已经缓存的列表，两次读取就可以显示一个特定的页面。  

当我们保存页面 `articles/1.html` 供离线使用时，我们这样处理。

    // Get the page content
    var pageHtml = document.body.innerHTML;
    var urlPath = location.pathName;
    // Save it in local storage
    localStorage.setItem( urlPath, pageHtml );
    // Get our index
    var index = JSON.parse( localStorage.getItem( index' ) );
    // Set the title and save it
    index[ urlPath ] = document.title;
    localStorage.setItem( 'index', JSON.stringify( index ) );

然后当用户离线情况下访问 `articles/1.html` 的时候，会得到 fallback.html，就像下面做的这样。  

    var pageHtml = localStorage.getItem( urlPath );
    if ( !pageHtml ) {
        document.body.innerHTML = `'<h1>Page not available</h1>'`;
    }
    else {
        document.body.innerHTML = localStorage.getItem( urlPath );
        document.title = localStorage.getItem( 'index' )[ urlPath ];
    }

我们也可以遍历 localStorage.getItem( 'index' ) 来获得用户缓存的所有页面的详细信息。  

### 同时使用 ApplicationCache 以及 LocalStorage  
[这里是个以上操作的 Demo](http://appcache-demo.s3-website-us-east-1.amazonaws.com/localstorage-cache/)。文章页可以通过页面右上方的按钮进行缓存，index 页面会显示出哪些页面可以在离线状态下使用。  

所有的代码[可以从 Github 获得](https://github.com/jakearchibald/appcache-demo/tree/master/www/localstorage-cache)。任何页面可以通过调用 [offliner.cacheCurrentPage()](https://github.com/jakearchibald/appcache-demo/blob/master/www/localstorage-cache/js/offliner-v1.js#L62) 来缓存。函数在每次进入 index 以及用户希望缓存的页面时被调用。  

如果用户遇到了 fallback 页面。 [offliner.renderCurrentPage()](https://github.com/jakearchibald/appcache-demo/blob/master/www/localstorage-cache/js/offliner-v1.js#L74) 会被调用来渲染指定的页面。如果没有页面可以被显示，就会显示错误信息。哦，这提醒了我....  

## 坑#7：我们不知道怎么获得 Fallback 页面  
当我们不能显示一个具体的页面的时候，我们可以显示的信息是[非常不清晰](https://github.com/jakearchibald/appcache-demo/blob/master/www/localstorage-cache/js/offliner-v1.js#L84)的。根据规范，如果初始请求遇到以下情况 Fallback 页面会被使用："被重定向到了另外的来源，返回了 4XX 或 5XX 或者存在网络错误（不包括用户取消了下载）"。  

某种程度上这不错。如果用户在线使用，但是网站挂掉了，浏览器只是会使用缓存的数据，用户甚至不会被通知。不幸的是，我们没办法提供用户使用 fallback 的原因。可能因为用户没有连接，可能是他们输入了不正确的 url，也可能是服务器报错。我们并不知道到底发生了什么。  

接下来让我们看下重定向。

## 坑#8：重定向到其余网站的请求会被认为失败  
是的，又是一个坑，好高兴啊你还在读。如果你想要去把自己锁在厕所的隔间，直到没有网络的时候才出来，我表示完全可以理解你。  
如果你的一个链接需要跳转到 Twitter 或者 Facebook 去做一些认证，我们友善的 Application Cache 会认为这是**不被允许的**，然后显示 Fallback 页面。  

这个规则还是有好的方面的。如果用户想要访问你的网站，但是他们使用的 wifi 强制他们重定向到 http://rubbish-network/pay-for-wifi-access (付费使用 wifi 的链接)，这个时候显示我们网站的 Fallback 页面是很好的。  

在需要验证的重定向的时候，NETWORK 段下的白名单并不能帮上什么。作为替代方案，你可以使用 JavaScript 或者 meta-redirect。呃。。  

### LocalStorage 方式的缺点  
“我们到了缺点的部分了吗？那刚刚说过的那些算什么啊？”，我知道，但是别烦我。单纯的 ApplicationCache 的解决方案有一些缺点。  

需要 JavaScript，虽然 Sprite Cow 使用 ApplicationCache 没有依赖 JavaScript，但是其余的网站都有使用。我以项上人头保证，很少人支持 ApplicationCache 但是不支持 JavaScript 。这并不是说 no-JavaScript 的支持不重要。[Lanyrd’s mobile 网站](http://m.lanyrd.com/)在浏览器不支持 JavaScript 的情况下依然运行的很好，实际上我们在旧的设备上避免了依赖 JavaScript 来使它变的简单和迅速。  

有些请求的体验并不好。FALLBACK 的问题是，在它之前原始链接必须已经失败，请求的发送和接受会消耗一定的时间。这样看 坑#1 (文件总会从 ApplicationCache 加载)，确实是蛮有用的。  

Opera 下 Fallback 不工作。Opera 不支持 manifest 属性中的 FALLBACK 段。希望他们能尽快的修复这个问题。  

### m.lanyrd.com 有什么不同点呢  
[之前看到的 Demo](http://appcache-demo.s3-website-us-east-1.amazonaws.com/localstorage-cache/) 只是我在 lanyrd 所做的事情的简化版。我们在 LocalStorage 中存储 JSON 数据，在 ApplicationCache 中存储对应的模版，而不是直接存储 HTML 文件供离线使用。这意味着我们可以不关心数据，只更新模版，也可以在不同的模版中使用相同的数据。  

我们的每个会议的 JSON 有个版本号。版本号在浏览网站的时候会被检测。如果他们与已经缓存的不匹配，则会更新。这意味着只在存在更新的时候发送更新请求。  

我们在用户跟踪或者下载事件发生时缓存数据，而不是提供按钮使用户可以离线存储特定的页面。这样浏览器可以知道用户需要哪些数据离线使用，这样如果他们更换了设备或者莫名其妙丢失了缓存，我们可以迅速的重新填充。  

通过 XMLHttpRequest 和 pushState 来切换页面。这在 Mobile 设备上更快，因为它不需要在每个页面加载的时候重新执行 JavaScript，也使网站更像一个 App。  

继续，由于遗留问题...
## 坑#9：关于 XHR 额外的一点  
离线情况下你可以发起 XHR 请求请求缓存资源，不幸的是，旧版本的 Webkit 以状态码为 0 返回这个请求，这种情况在流行的库中会被认为请求失败。  

    // In jQuery...
    $.ajax( url ).done( function(response) {
        // Hooray, it worked!
    }).fail( function(response) {
        // Unfortunately, some requests
        // that worked end up here
    });

具体点，这个问题存在于 Blackberry Playbook 和 iOS3 设备以及 Android 3/4。Android 2 没有这个 Bug，奇怪的是，它看上去运行了新版的 Webkit 内核，它支持 `history.pushState ` 而最新版的 Android 并不支持。他X的 ANDROID，以下是你需要处理这个问题的方式。  

    $.ajax( url ).always( function(response) {
    // Exit if this request was deliberately aborted
    if (response.statusText === 'abort') { return; }
    // Does this smell like an error?
    if (response.responseText !== undefined) {
        if (response.responseText && response.status < 400) {
            // Not a real error, recover the content
            response = response.responseText;
        }
        else {  
            // This is a proper error, deal with it
            return;
        }
    }
    // do something with 'response'
});

你可以在[这个 Demo](http://appcache-demo.s3-website-us-east-1.amazonaws.com/localstorage-cache/) 上进行测试。

### ApplicationCache: 你的朋友，你的坑
我没有说不应该使用 ApplicationCache，它是很有用的。我们都知道有些人需要特别提醒，或者需要特殊的“看管”来保证他们不做蠢事。嗯，ApplicationCache 就是其中一个。  

在细心的照料下，ApplicationCache 能做一些其他人不能做的事情。但是当他对你说“你不用雇佣水管工，我自己就可以搞定那整个浴室，我曾经搞定了白金汉宫所有的浴室，你看...”的时候，请绅士的拒绝他。  

如果你不是在创建一个不需要任何网络请求的完全客户端的应用的话，请使用尽可能小的 ApplicationCache，其余的事情交给 LocalStorage 吧，这样会减少很多的麻烦。