---
layout: post
category : front-end
title: 使用 Application Cache
date: 2012-11-08 00:12:02
tags : [Application Cache, AppCache, 使用]
---
 
原文链接地址 [Using the application cache (MDN)](https://developer.mozilla.org/en/Offline_resources_in_Firefox)

## 简介  

HTML5 提供了一套可以让 web 应用离线运行的机制，开发者可以使用 application cache（AppCache）接口定义浏览器需要缓存的供离线用户使用的资源。即便是用户在离线的情况下刷新页面，被缓存的应用都应该被正确的加载和运行。  
使用 AppCache 可以给应用带来如下的好处：

* 离线浏览：即便是在离线的情况下用户都可以访问网站。  
* 速度提升：被缓存的资源是本地的，所以可以加载的更快。  
* 减小服务器负载：浏览器只会加载服务器端已经更新的资源。

<!-- more -->

## AppCache 的工作方式
### 使用 AppCache
想要在应用中使用 AppCache，需要在应用页面的 *html* 节点上增加 *manifest* 属性，如下所示：  

    <html manifest="example.appcache">  
    ...   
    </html>
*manifest* 属性链接到一个 **cache manifest** 文件，文件中列出了应用中需要缓存的资源列表。  
*manifest* 属性需要包含在每一个需要缓存的页面上。除非被显式的声明，浏览器不会缓存不包含 *manifest* 属性的页面。在 manifest 文件中不需要列出所有希望缓存的页面（pages），对于包含了 *manifest* 属性的页面，浏览器会隐式的将其缓存到 AppCache 中。  

### 加载文档
使用 AppCache 会影响加载 document 的流程。    

详细的加载主文档以及更新 AppCache 的过程如下所示：  

1. 当浏览器访问一个包含了 *manifest* 属性的页面时，如果 AppCache 不存在，浏览器会加载主文档，然后获取所有在 manifest 文件中列出的资源，创建 AppCache 的第一个版本。
2. 随后访问页面，从而浏览器会从 AppCache 中加载主文档以及其余的在 manifest 文件中定义的资源（不是从服务器获取）。此外，浏览器会向 [window.applicationCache](https://developer.mozilla.org/en/DOM/window.applicationCache) 对象发送一个 *checking* 事件，通过 HTTP 请求获取 manifest 文件。
3. 如果当前的 manifest 文件是最新的，浏览器发送一个 *noupdate* 事件给 [window.applicationCache](https://developer.mozilla.org/en/DOM/window.applicationCache) 对象，更新结束。需要强调的是，如果在服务器端更新了任何缓存中的资源文件，必须同时更新 manifest 文件，以保证浏览器知道需要重新获取所有资源。
4. 如果 manifest 文件已经更新，所有 manifest 文件中列出的文件（包括通过调用  [applicationCache.add()](https://developer.mozilla.org/en/nsIDOMOfflineResourceList#add.28.29) 添加的资源）都会遵循基础的 HTTP 缓存规则，被加载到一个临时缓存中。对于每一个加载到临时缓存区的文件，浏览器会发送一个 *progress* 事件给 *applicationCache* 对象，如果过程中出现了任何错误，浏览器会发送一个 *error* 事件，停止更新的过程。
5. 当浏览器收到所有的文件之后，他们自动会被移动到一个实际的离线缓存区，同时 *cahced* 事件被发送给 *applicationCache* 对象。由于主文档已经被从缓存中加载到浏览器中，已经更新的文档除非重新加载（包括人工或者程序内部）都不会被重新渲染。

### 缓存存储位置以及 AppCache 的清除

在 Chrome 中，可以通过点击 “preferences” 下的 “Clear browsing data...” 或者访问 [chrome://appcache-internals/](chrome://appcache-internals/) 来清除离线缓存。Safari 在 “preferences” 中有一个类似的 “Empty cache” 设置项，但是设置之后需要重启浏览器。

AppCache 同样可以过时，如果应用的 manifest 文件被从服务器端删除，浏览器会删除使用 manifest 的所有应用缓存，同时发送 *obsoleted* 事件给 *applicationCache* 对象。AppCache 的状态被设置为 OBSOLETE。

## manifest 文件

### 引用一个 manifest 文件

web 应用中的 *manifest* 属性，可以通过相对路径和绝对路径（需要和 web 应用在同一个域下）两种方式进行。manifest 文件可以有任意的后缀，但是 MIME type 必须是 *text/cache-manifest*。

### manifest 文件中的条目

manifest 文件是一个列出了浏览器需要缓存的文件的简单文本文件。资源通过 URI 进行识别。manifest 中列出的条目必须和 manifest 具有相同的域名和端口。

### 示例1：一个简单的 manifest 文件
下面是一个简单 manifest 文件示例， 示例网站 www.example.com 下的 *example.appcache*。

```
CACHE MANIFEST  
# v1 - 2011-08-13  
# This is a comment.  
http://www.example.com/index.html    
http://www.example.com/header.png  
http://www.example.com/blah/blah  
```
 
manifest 文件可以包含三段 （CACHE、NETWORK 和 FALLBACK）。在上面的示例中，没有分段，所以所有的列出的行都会被放到默认的段（CACHE）中去，表示浏览器应该在 AppCache 中缓存所有的资源。

示例中的 “v1” 存在的原因如下：浏览器仅在 manifest 文件变化后才会更新 AppCache，如果更新了一个缓存了的资源（例如，更新了缓存中的 *header.png* 文件），必须更新 manifest 文件，从而让浏览器知道它需要更新缓存。对于 manifest 文件的所有修改都是可以的，但是推荐使用修改版本号的形式实现。  
**注意：不要在 manifest 文件中缓存自身，否则想要通知浏览器 manifest 更新是几乎不可能的**

### manifest 文件中的三段 CACHE，NERWORK 以及 FALLBACK
一个 manifest 文件可以包含三个分段：CACHE，NETWORK 和 FALLBACK。

CACHE：  
这是 *manifest* 文件中条目的默认分段，在 CACHE 段中列出的文件在第一次下载后会被缓存。  
NETWORK：  
在 NETWORK 段下的文件建立了需要建立网络连接的白名单。即便是用户离线的情况下，所有此段下的资源会绕过缓存。可以使用通配符。  
FALLBACK：  
FALLBACK 段下的内容定义了资源不可用的情况下的备用资源。所有此段下的条目包含两个 URI，第一个是资源链接，第二个是备用资源链接。所有的 URI 同样不可以跨域。通配符也是可用的。

CACHE，NETWORk 和 FALLBACK 三个段在 manifest 文件中出现的顺序是随意的，并且每个段在文件中出现的次数没有限制。

### 示例2：一个更加完整的 manifest 文件
以下是一个假象网站 www.example.com 下的更加完整点的 manifest 文件。  

```
CACHE MANIFEST  
# v1 2011-08-14  
# This is another comment 
index.html  
cache.html  
style.css  
image1.png

# Use from network if available  
NETWORK:
network.html

# Fallback content  
FALLBACK:   
/ fallback.html  
```

示例使用了 NETWORK 和 FALLBACK 段，定义了 `network.html`  必须从直接从服务器获取，`fallback.html` 备网站链接 / 无法访问的情况下访问。

### manifest 文件结构
manifest 文件必须以 text/cache-manifest 的 MIME type 返回。  
manifest 文件是一个 UTF-8 编码的文本文件，可以包含 BOM 头。文件第一行必须包含 ‘CACHE MANIFEST’ （中间以空格隔开）字符串，MANIFEST 后需要接一个或者多个空白字符，所有其他文本都会被忽略。  
文件中的其余部分必须由以下各行组成：  

- **空白行**  
可以使用包含任意多个空格或 Tab 的空白行  
- **注释**  
注释行需要以 # 加任意多个空白字符开始，后面可以包含任意字符。只能注释单行，不能注释一个片段。  
- **段头**  
共有三个可选的段头取值：  
CACHE：   切换到缓存段。  
NETWORK： 切换到白名单段。  
FALLBACK：切换到备用段。  
段头可以包含空包字符，但是必须包含 “:” 。
- **规则**  
行的格式决定于到底属于哪个段，在 CACHE 段中，每一行是指向需要缓存的资源的 URI 或者 IRI （不能包含通配符）。行的开头和结尾处可以存在空白符。FALLBACK 行包含两个指向资源的 URI 或者 IRI。NETWORK 段中的内容同样包含一个 URI 或 IRI （可以使用通配符 * ）。
**注意：相对 URI，相对于 manifest 文件的 URI，而不是包含 manifest 的 document**  

## AppCache 中的资源
一个 AppCache 通常包含一个资源，通过 URI 识别。
**注意：资源可以属于多个分类，比如可以同时属于 CACHE 和 FALLBACK**  
资源分类的详情如下：  

- **Master Entries**
Master Entries 是任何在 `<html>` 标签中包含了 manifest 属性的  HTML 文件，例如有一个 HTML 文件， http://www.example.com/entry.html，代码如下：

```
<html manifest="example.appcache">  
  <h1>Application Cache Example</h1>  
</html>  
```

如果 entry.html 并没有在 *example.appcache* 中列出来，访问 entry.html 时，会将其自身作为 Master Entries 加入到 AppCache 中。

- **Explicit Entries**  
Explicit Entries 是在 manifest 文件中的 CACHE 段中列出的资源。

- **Network entries**
manifest 文件中的 NETWORK 段中包含 web 应用需要联网的资源，Network entries 实际上是需要请求服务器而不访问缓存的白名单  
**注意：在 manifest 文件中简单的覆盖 Master Entries 导致的结果是不确定的，因为 Master Entries 是在后续访问页面时被添加到 AppCache 的**

- **Fallback entries**  
Fallback entries 的用处在资源加载失败时。

## 缓存状态

每个 AppCache 都会有一个状态，用来标示缓存的当前状态，用来相同 manifest URI 的缓存，享有同样的状态，所有的状态列表如下：

- **UNCACHED**  
特殊值，表明 AppCache 还未被完全初始化。  
- **IDLE**  
AppCache 当前不在被更新的过程中。  
- **CHECKING**  
manifest 文件正在被加载或者检查是否已更新。  
- **DOWNLOADING**  
资源正在被下载到缓存中，会导致 manifest 中的资源更新。  
- **UPDATEREADY**  
存在更新的 AppCache，当新的缓存已经被下载但是没有使用 swapCache 方法更新的时候，相应的会触发 *updateready* 事件，而不是 *cached* 事件。  
- **OBSOLETE**  
AppCache 已经过时。  

## 检测 manifest 文件是否已经更新

可以在程序中使用 JavaScript 检测 manifest 文件是否已更新，由于有可能在脚本添加更新的事件监听之前 manifest 文件已经更新，所以脚本应该检查 `window.applicationCache.status`。 

```
function onUpdateReady() {  
  alert('found new version!');  
}  
window.applicationCache.addEventListener('updateready', onUpdateReady);  
if(window.applicationCache.status === window.applicationCache.UPDATEREADY) {  
  onUpdateReady();  
}  
```
    
需要手动测试新的 manifest 文件时，可以使用 `window.applicationCache.update()` 方法。

## 需要注意的点


- 永远不要通过增加 GET 参数的方式（例如：`other-cached-page.html?parameterName=value`）获取缓存的文件，这会导致浏览器忽略缓存直接从网络获取文件。如果需要链接到有参数传递的资源，在 hash 上增加参数，例如：`other-cached-page.html#whatever?parameterName=value`。  
- 当应用被缓存后，简单的更新资源文件是不够的。必须手动更新 manifest 文件，从而让浏览器使用新的文件。可以在程序内通过 `window.applicationCache.swapCache()` 方法来更新文件，但是已经被加载的资源不会受影响。
- 服务器端设置 `*.appcache` 文件的 expires headers 为立即过期是个很好的做法，这样可以避免 manifest 文件被缓存带来的风险。例如，在 Apache 中可以添加如下设置：  
`ExpiresByType text/cache-manifest "access plus 0 seconds"`

