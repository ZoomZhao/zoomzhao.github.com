---
layout: post
category : front-end
title: Medium LESS 编码指引
date: 2015-07-30 16:27:49
tags : [Medium, CSS]
---

# Medium LESS 编码指引

Medium 使用 [LESS](http://lesscss.org/) 的一个严格子集来进行 CSS 预处理。严格子集只包括变量和 mixin，不包含任何其它内容（嵌套等）。

Medium 的命名约定改编自 SUIT CSS 框架。具体的说，它依赖于 _结构化的类名_ 和 _有意义的分隔符_ （分隔符不仅仅用来分隔单词）来约定命名。这样的设计可以帮助拟补当前给 DOM 设置 CSS 的不足（例如，缺少样式封装），同时可以改善不同 class 之间的联系。

<!-- more -->

**Table of contents**

- [JavaScript](#javascript)
- [Utilities](#utilities)
    - [u-utilityName](#u-utilityName)
- [Components](#components)
    - [componentName](#componentName)
    - [componentName--modifierName](#componentName--modifierName)
    - [componentName-descendantName](#componentName-descendantName)
    - [componentName.is-stateOfComponent](#is-stateOfComponent)
- [Variables](#variables)
    - [colors](#colors)
    - [z-index](#zindex)
    - [font-weight](#fontweight)
    - [line-height](#lineheight)
    - [letter-spacing](#letterspacing)
- [Polyfills](#polyfills)
- [Formatting](#formatting)
    - [Spacing](#spacing)
    - [Quotes](#quotes)
- [Performance](#performance)
    - [Specificity](#specificity)

<a name="javascript"></a>

## JavaScript

语法: `js-<targetName>`

JavaScript 特有的 classes 可以减少改变组件的结构或者样式时不经意间影响关联的 JavaScript 行为的风险。它只是个工具，并不是所有情况下都必要。如果你需要创建一个并不会对它添加样式，而只是 JavaScript 中的一个选择器的 class，你就应该添加 `js-` 前缀。实际使用，会像下面这样：

``` html
<a href="/login" class="ban ban-primary js-login"></a>
```

**再次强调，JavaScript 特有的 classes 在任何情况下，都不应该被设置样式。**

<a name="utilities"></a>

## Utilities

Medium 的工具 classes 包括一些低层次的结构和位置。工具类可以直接作用到任意的元素上；多个工具类可以同时使用；并且工具类可以和组件类同事使用。

工具类的存在是因为特定的 CSS 属性和模式很常用。比如：浮动，内容浮动，垂直居中，文本截断等。通过工具类可以去重并且提供一致的实现。它们同时充当一个功能性（比如： non-polypill） mixin 的替代品。


``` html
<div class="u-clearfix">
  <p class="u-textTruncate">{$text}</p>
  <img class="u-pullLeft" src="{$src}" alt="">
  <img class="u-pullLeft" src="{$src}" alt="">
  <img class="u-pullLeft" src="{$src}" alt="">
</div>
```

<a name="u-utilityName"></a>

### u-utilityName

语法: `u-<utilityName>`

工具类必须使用驼峰式命名法，使用 `u` 作为命名空间前缀。以下展示一个示例，怎样在一个组件中，使用不同的工具类建立一个简单的结构。

``` html
<div class="u-clearfix">
  <a class="u-pullLeft" href="{$url}">
    <img class="u-block" src="{$src}" alt="">
  </a>
  <p class="u-sizeFill u-textBreak">
    …
  </p>
</div>
```

<a name="components"></a>

## Components 组件

语法: `<componentName>[--modifierName|-descendantName]`

组件驱动开发为阅读和编写 HTML CSS 代码提供了很多好处：

- 帮助区分组件 root 节点，子节点，以及修改器的 class。
- 保持底层选择器的唯一性
- 帮助从文档语义中分离展现语义

你可以认为组件是一个封装了语义，样式和行为的特殊元素。


<a name="componentName"></a>

### ComponentName

组件名称必须使用驼峰式命名法。

``` css
.myComponent { /* … */ }
```

``` html
<article class="myComponent">
  …
</article>
```

<a name="componentName--modifierName"></a>

### componentName--modifierName

组件修改器的作用是在某些方面修改基础组件的展现。修改器必须使用驼峰式命名法，并且使用两个分隔符与组件名称分割。修改器 class 必须在组件 class _之外_ 被添加到 HTML 中。

``` css
/* Core button */
.btn { /* … */ }
/* Default button style */
.btn--default { /* … */ }
```

``` html
<button class="ban btn--primary">…</button>
```

<a name="componentName-descendantName"></a>

### componentName-descendantName

一个组件后代是一个附加到一个组件子元素上的 class。它负责给特定组件的子元素提供展现。组件后代必须以驼峰式命名法命名。

``` html
<article class="tweet">
  <header class="tweet-header">
    <img class="tweet-avatar" src="{$src}" alt="{$alt}">
    …
  </header>
  <div class="tweet-body">
    …
  </div>
</article>
```

<a name="is-stateOfComponent"></a>

### componentName.is-stateOfComponent

使用 `is-stateName` 来给状态相关的组件修改器命名。状态名称必须使用驼峰式命名法。**永远不要直接给这些 class 定义样式；它们应该始终被当作辅助 class 使用。**

JS 可以添加/删除这些 classes。这意味着相同的状态名称可以在不同的上下文同时使用，但是每个组件必须给状态定义自己的样式。

``` css
.tweet { /* … */ }
.tweet.is-expanded { /* … */ }
```

``` html
<article class="tweet is-expanded">
  …
</article>
```



<a name="variables"></a>

## Variables 变量

语法: `<property>-<value>[—componentName]`

我们的 CSS 中变量名也是严格限制格式的。这种语法为属性，使用和组件中提供了强关联。

下面的变量定义是一个颜色属性，使用 grayLight 颜色，使用到 highlightMenu 组件中。

``` CSS
@color-grayLight—highlightMenu: rgb(51, 51, 50);
```

<a name="colors"></a>

### Colors

当实现样式属性值时，你只可以使用 colors.less 提供的颜色变量。

向 colors.less 中添加颜色变量时，优先使用 RGB 和 RGBA，而不是 hex，名称，HSL 或者 HSLA。

**Right:**

``` css
rgb(50, 50, 50);
rgba(50, 50, 50, 0.2);
```

**Wrong:**

``` css
#FFF;
#FFFFFF;
white;
hsl(120, 100%, 50%);
hsla(120, 100%, 50%, 1);
```

<a name="index"></a>

### z-index scale

请使用 z-index.less 文件中限制的 z-index 取值。

`@zIndex-1 - @zIndex-9` 可供选择。不应该有取值超过 `@zIndex-9`。



<a name="font weight"></a>

### Font Weight

由于 web fonts 的支持，`font-weight` 的角色比原来更加重要了。不像原来 `bold` 是个可以加粗文字的算法，现在不同的字重会渲染对应的不同文字。当然，使用 `font-weight` 的数值取值可以强化文本的展现。下面是一些指引。

直接定义字重应该被避免，使用合适的字体变量：`.font-sansI7, .font-sansN7, 等等`

后缀定义了字重和样式：

``` CSS
N = normal
I = italic
4 = normal font-weight
7 = bold font-weight
```

查看 type.less 来定义 type size, letter-spacing 和 line height。

Raw sizes, spaces, and line heights 取值应该从 type.less 中选择。



``` CSS
ex:

@fontSize-micro
@fontSize-smallest
@fontSize-smaller
@fontSize-small
@fontSize-base
@fontSize-large
@fontSize-larger
@fontSize-largest
@fontSize-jumbo
```

See [Mozilla Developer Network — font-weight](https://developer.mozilla.org/en/CSS/font-weight) for further reading.



<a name="line height"></a>

### Line Height

Type.less 同时提供了行高标尺。它应该被用于块状文本。

``` CSS
ex:

@lineHeight-tightest
@lineHeight-tighter
@lineHeight-tight
@lineHeight-baseSans
@lineHeight-base
@lineHeight-loose
@lineHeight-looser
```

Alternatively, when using line height to vertically center a single line of text, be sure to set the line height to the height of the container - 1.

``` CSS
.btn {
  height: 50px;
  line-height: 49px;
}
```

<a name="letterspacing"></a>

### Letter spacing

文本间距同样需要从以下变量标尺中选择。

``` CSS
@letterSpacing-tightest
@letterSpacing-tighter
@letterSpacing-tight
@letterSpacing-normal
@letterSpacing-loose
@letterSpacing-looser
​````

<a name="polyfills"></a>
## Polyfills

mixin syntax: `m-<propertyName>`

在 Medium 我们只使用 mixin 来生成浏览器前缀 polyfills。

border radius mixin 示例:

​```css
.m-borderRadius(@radius) {
  -webkit-border-radius: @radius;
     -moz-border-radius: @radius;
          border-radius: @radius;
}
```



<a name="formatting"></a>

## Formatting

以下是高层次页面构建样式规则。

<a name="spacing"></a>

### Spacing

CSS 规则需要逗号分割并另起一行：

**Right:**

``` css
.content,
.content-edit {
  …
}
```

**Wrong:**

``` css
.content, .content-edit {
  …
}
```

CSS 块需要间隔一行，不可以是两行也不可不隔行。

**Right:**

``` css
.content {
  …
}
.content-edit {
  …
}
```

**Wrong:**

``` css
.content {
  …
}

.content-edit {
  …
}
```



<a name="quotes"></a>

### Quotes

引号在 CSS 和 LESS 是可选的。我们使用双引号让视觉上更清楚的区分字符串和选择器或者样式属性。

**Right:**

``` css
background-image: url("/img/you.jpg");
font-family: "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial;
```

**Wrong:**

``` css
background-image: url(/img/you.jpg);
font-family: Helvetica Neue Light, Helvetica Neue, Helvetica, Arial;
```

<a name="performance"></a>

## Performance

<a name="specificity"></a>

### Specificity

虽然层叠式存在与名字（层递式样式表）中，它带来了非不要的性能问题。参照以下示例：

``` css
ul.user-list li span a:hover { color: red; }
```

样式在页面渲染 layout 的过程中解析。选择器自右向左解析，在选择器无法适配时退出。因此，在上面的示例中，每一个 a 标签都会被检查是不是处在一个 span 和 li 中。就像想象中的一样，这需要一系列的 DOM 遍历，对于大型文档来说可以带来显著的 layout 耗时增加。访问 [https://developers.google.com/speed/docs/best-practices/rendering#UseEfficientCSSSelectors](https://developers.google.com/speed/docs/best-practices/rendering#UseEfficientCSSSelectors) 查看更多信息。

如果只是想在 `.user-list` 下的所有 `a` 元素 hover 时变红，可以将样式简化为：

``` css
.user-list > a:hover {
  color: red;
}
```

如果你只想给予 `.user-list` 下的部分 `a` 元素一些样式，可以给它们一个特定的 class：

``` css
.user-list > .link-primary:hover {
  color: red;
}
```
