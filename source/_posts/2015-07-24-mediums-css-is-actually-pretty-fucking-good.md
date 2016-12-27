---
layout: post
category : front-end
title: Medium 的 CSS 真他*的好
date: 2015-07-23 21:34:10
tags : [Medium, CSS]
---

# Medium 的 CSS 真他*的好

- 翻译自 [Medium by fat](https://medium.com/@fat/mediums-css-is-actually-pretty-fucking-good-b8e2a6c78b06)

> I always believe that to be the best, you have to smell like the best, dress like the best, act like the best. When you throw your trash in the garbage can, it has to be better than anybody else who ever threw trash in the garbage can.   
>
> — Lil Wayne

一段时间以来，我都想要写一些关于 Medium CSS 的东西。关于 Medium 的 CSS 有什么样的故事？我们做了哪些不一样的事情？怎么才能像我们一样做，或者可以从我们这学到点什么？

以下包括一些 CSS 的记录，包括我们是怎么一步步走过来的，以及现状。

<!-- more -->

## 开始（一些历史）
大概2年前，我加入了 Obvious Corp. 参与 *medium.com* 的开发工作。

当时，Medium 已经进行了一系列的 “re-styles” (re-style 就是设计师通过一些比之前漂亮的东西来摧毁你的生活，结果就是，你要重写成吨的 CSS/LESS/SASS/等等)。由于这些 re-styles，代码中出现了非常多令人讨厌的东西 — 非常倚重 LESS 的语言特性，页面驱动的语义，non-sprited/non-retina 的图片资源，等等。

我翻看 Git 以及其他代码管理工具获取到了 Medium 2012 年的个人资料页面的实现，*不寒而栗*。下面列出了发现的问题，请注意那些非常不应该的模式：

- 所有样式都嵌套在 `.profile-page` class 下，没有任何重用组件（另外 *几乎* 所有样式都使用了 `.profile` 前缀）
- 超级通用变量名形如 `@link-color` 只在 `profile-page.less` 中被使用，并没有其他用处
- 非常深的嵌套（`.profile-page .profile-posts .home-post-summary .home-post-collection-image img` 是一个真实存在的选择器 - 性能会非常差）
- 没有 image spriting
- 没有 z-index 标准，没有 font 标准，没有 color 标准，没有 ** 标准

[Old medium profile page](https://gist.github.com/fat/a4af78882d0003d2345e)

## Project 1：图片
当时，我参与了很多库 (Bootstrap, Ratchet, 等等)的开发工作，仔细处理每一个细节，希望能写出最好的 CSS 代码。

Medium 的 CSS 代码当然非常不一样，不是简单的不同，而是屎一样的烂。我想解决这个问题。

了解了所有需要处理的东西之后，我选择了 images 开始下手。我记得在了解到我们直到 2012 年没有做任何的 spriting 之后非常震惊。大约数百个图片资源，例如 placeholders, arrows, icons 等等，存在于 /img/ 文件夹中，简直是 icons 坟墓。

为了解决这个问题，我做了两件事情。第一，我写了一个 CSS 工具脚本叫做 SUS（我们现在还在使用它，我将它开源在这里：[https://github.com/medium/sus](https://github.com/medium/sus)）,这个东西大部分是在 Guillermo Rauch 的帮助下用 IRC 编写完成，作用是从样式表中分离出图片，然后在另一个包含 data-uri 的文件中 lazy 加载。这只是一个小工具来帮助我们认识到怎么做。

第二件事情，我和 Geoff Teehan 坐下来，创建了 Medium 的第一个 icon font。当时我们并不清楚具体要怎么做。经过几个漫长的夜晚，在 icomoon 和一个 Github 文章的帮助下，我们产出了一个对 Medium *非常* 好的东西。说它非常好，是因为我们可以删除 img 文件夹中 97% 的文件，每一个图标在我的 retina MacBook Pro 上都显示的非常好，并且只需要请求非常少的资源。

## Project 2: 标准
对我来说，另外一个大的工程是 z-index 标准。z-index 是个很容易就会失控到让人抓狂的东西。我不想让这种事情也出现在 Medium。 

项目开始之前，很容易看到一个元素 `z-index:5;` 跟在一个 `z-index: 1000000;` 的节点后面，另外一个兄弟节点的样式是 `z-index: 1000001;`，还有一个 `z-index: 99999;`。

在代码库这样的样式中比比皆是，因为并没有明确的方式指明应该怎么做。

我开始了一项在整个代码库中人工审核 z-index 取值的艰巨任务，然后引入了一个可以被 z-indexing 组件使用（限制 1-10）的标准（z-index.less）。最终将所有的 z-index 定义都改用 z-index.less，这样就可以看出不同组件的相对位置了（实际上这是很方便的）。

下面是当前 medium.com 使用的 z-index 文件。

[https://gist.github.com/fat/1f6da6b3bd0311a1f8a0](https://gist.github.com/fat/1f6da6b3bd0311a1f8a0)

当然，color 标准（黑色，灰色，logo 颜色），type 规范（字体大小，字重，字间距，行高）紧随 z-index 之后需要制定。

另外，你可能注意到，变量命名遵循了一定的规范，并且采用了语义化命名，稍后会详细介绍。

## Project 3: 新的样式指引

制定了 Medium 规范后不久，我们开始了一场大的代码重构，代号 “Cocoon”。

Cocoon 抛弃了一些 post 模版（Medium 本来包含图片模版，引用模版等，不仅仅包括单个文章的模版），并且使用 post 列表替代了 post “cards”。

我们将这次重构作为重新思考 Medium 并创建一个新的样式指引的机会。

最初由我，Dave Gamache，Dustin Senos，和 Chris Erwin 共同来完成了这项工作。

我们花了一些时间来着实加强我们在 Medium 编写生成环境 CSS/LESS 的想法。主要的更新如下：

- 限制 LESS 的使用，只允许使用 variables 和 mixins (不允许使用 nesting, guards, extend, 等等) ，虽然其它的语言特性 *可以* 非常强大，但是我们发现经验没那么丰富的 LESS 开发者很容易就会陷入麻烦。我们同时选择了纯 CSS 的视觉审美（也包括它提供的一致性）并且想要我们的代码库向着这样的方向发展。
- Classes 和 IDs 都是小写字母，中划线分割单词－这正是我们在 Bootstrap, Skeleton, Ratchet, 等等定义选择器的方式，我们认为我们在这里也可以这样做。另外一个原因是，这种定义方式遵循了 CSS 语言自身的定义方式，比如：border-radius-top-left。
- 选择 components 而不是 page 层面的样式－我们希望我们的前端更像是一个库，根据这样的想法，我们将 profile.less 拆分成更加具体的文件，例如：button.less, dialog.less,tooltip.less 等。

以下是完整的样式指引：
[https://gist.github.com/fat/b27700946c777adacdf4](https://gist.github.com/fat/b27700946c777adacdf4)

这并不完美，但是它理清了一些基础的思路，然后带领我们向着感觉上正确的方向前进。

不幸的是，在样式指引更新之后，人们依然非常纠结，什么时候需要定义组件，什么时候需要定义子组件等等。并且我们偶尔会碰到一些并不很具体的东西，例如 classname：.nav-on-light-background-button 或者 .button-primary-sidebar-over-blur.

现在人们不再使用 page 级别的前缀来定义 class 了（这是很好的），但是它们开始使用中划线间隔拼接很多任意单词，组成 classname。进化的过程如下：.button → .button-primary → .button-primary-dark → .button-primary-dark-container → .button-primary-dark-container-label，真是让人不舒服。

## Project 4: 确定未来
当时，我开始以成为“世界上最好代码”为目标在 Medium 内部写了 *很多* 的 CSS 代码，但并不是非常清楚最好的 CSS 代码到底是什么样子，但至少明白了当时的方向并不成功。

> 人们编写代码的过程中会感到很困惑，更糟糕的是，他们以为自己写的 CSS 非常好，实际上却不是这样。

所以，我开始四处观察－－了解一些框架，尝试不同的工具和哲学，与朋友沟通，与朋友的朋友沟通，等等。很快，我找到三个为了让我们的 CSS 代码向着正确的方向前进需要解决的问题。

1. 引入新的 CSS variable, mixin, 和 classname 语义来避免过长的 classnames 以及增强可读性。
2. 从 LESS 转到 Rework 来获得更强的 mixin support 和更佳接近于原生 CSS 的语法。
3. 引入 CSS 性能（加载时间，fps，layout time 等）工具，使跟踪样式修改和回归更加简单。

## Project 5: 语义
我想要给我们的代码库一个更加严格的语义化标准。因为对于我们这种大小的团队，我觉得拥有一个可以依靠的规则是更简单的。并且我宁愿让事情变得更复杂一点，尽管这也意味着人们必须在创建新的 classes 的时候要更加留神。不过不惜一切代价，我希望能够避免过长的 classname，或者至少没那么容易出现。

我开始就这个问题，与 Daryl Koopersmith 以及我的好朋友 Nicolas Gallagher 进行了长时间的讨论。

Nicolas 跟我的关系很有趣，Nicolas 经常告诉我一些事情，我说他是错的，叫着他的法语名字（Va te faire foutre, enculé），在他身边得瑟几个星期，直到不可避免的意识到他是对的，然后把他的想法划归己有。

这一次也是一样，经过几个漫长的夜晚，我终于完成了一个与 Nicolas 的 SUITCSS 很像的语义化样式。很像，但是又那么 ~一丢丢~ 方面更好。

于是，我开始抄袭 Nicolas 的东西，我扔掉了就的样式指引，然后复制粘贴了他的很大一部分，左改改，右改改。

最终，我产出了我们当前使用的最新的样式指引，你可以在下面完整的阅读它，主要的更新是：

- **.js-** 开头的 class names 代表依赖 javascript 选择器的元素
- **.u-** 开头的 class name 单独目的的工具类，例如 .u-underline, .u-capitalize, 等等
- 引入有意义的连字符以及驼峰－－区分 component, descendant components, 和 modifiers 的间隔符。
- **.is-** 开头的 classes 表示有状态的 classes (经常被 js 切换) 例如 .is-disabled
- 新的 CSS 变量语义： [property]-[value]—[component]
- mixing 限制为只用在 polyfills 并且以 **.m-** 开头

[https://gist.github.com/fat/a47b882eb5f84293c4ed](https://gist.github.com/fat/a47b882eb5f84293c4ed)

复制粘贴样式指引是一回事，而重写你的 CSS 又是另一回事。

庆幸的是我可以说服公司这个项目的重要性，然后他们给了 2½ 周的时间让我们重写 Medium.com 的所有 CSS－－如果你一直在关心我们的话，这只是我们[修复 CSS underline 问题](https://medium.com/designing-medium/crafting-link-underlines-on-medium-7c03a9274f9)所花费时间的很小一部分。

也就是说，这次重写不只是宣泄，我们清理了一些比没用的样式，收集并整理了整个网站的实现，将文件拆解成小的子模块。让我惊喜的是，只造成了五次以内的回滚。

当然，重构 CSS 意味着重构我们的模版－－增加更多的封装和严格的语义化。现在我们不再有数以百计使用随机 avatar classes 的一次性 <IMG> 标签，我们使用一个单独的集中式 avatar 模版，模版接受布尔选项，来触发不同的 modifiers 比如大小，样式等等。这让更新样式变的简单，而已实现细节时引入更少的 Bug。

## Project X: 未来？
毫无疑问，我们比两年之前变的更好了。在 Medium 写 CSS 变的非常愉快，不同经验的开发者们也在促进更加伟大的东西。

大家都说，未来的 CSS 项目必须是关注性能。随着我们不断的完善我们的故事，然后推向下一个阶段，你可以想象怎样精确可靠的测量 layout 和 渲染性能是非常重要的。

所以，还要更多的事情要做，但是我现在感觉我们已经非常好了，比我平时想的都要好。

为你完成了这个冗长又无聊的 post 干杯。

感谢 Katie, Dave, Mark, Koop, Kristofer, Nicolas 以及其它帮助我的人❤。