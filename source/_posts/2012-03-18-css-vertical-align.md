---
layout: post
title: 理解 CSS 的 vertical-align 属性
---

原文地址：[Understanding CSS’s vertical-align Property](http://www.impressivewebs.com/css-vertical-align/) by Louis Lazaris

你看，那个前端开发又在哭诉：“ X，Vertical-align 又不起作用了！”

`vertical-align` 属性看上去很简单，但是确实会给 CSS 新手带来很大的麻烦，即便是 CSS 的老手在这个点上也经常需要停下来把问题搞搞清楚。

接下来，我会尝试用一个容易理解的方式来介绍它。

## 什么是它不会做的 ##

对于 **vertical-align** 有一个很常见的误解是，当它作用于一个元素的时候，它会改变这个元素内部所有内容到指定的位置。例如：当你在一个元素上使用 **vertical-align: top**, 这个元素里面所有内容被期望置顶显示。

<!-- more -->

这让我想起使用 Table 时的情况：

```
<td valign="top">
  Whatever...
</td>
```

在 Table 中，“valign” 属性（在 HTML5 中被废弃）会让表格中的所有的元素置顶显示。所以，很自然的，当 CSS 开发人员使用 **vertical-align** 时会期望达到同样的效果，可惜的是，它只会在当前元素上起作用。

## 它会做些什么 ##
**vertical-align** 属性以下三种情况下发挥作用：

1. 它只在 inline 或者 inline-block 元素上有效。
2. 它只会影响元素自身的对齐方式，而不在它的内容上生效（表格单元格除外）。
3. 当它被应用于表格的单元格时，影响表格内容的对齐方式，而不是单元格自身。

所以，下面的代码不会产生任何的影响:

```
div {
  vertical-align: middle; /* this won't do anything */
}
```

为什么呢？因为 `<div>` 是 `block-level` 元素，而不是 `inline`。当然如果你改变 `div` 到 `inline` 或者 `inline-block`, **vertical-align** 就会生效了。

也就是说，如果正确使用（在 `inline` 或者 `inline-block` 元素上使用），**vertical-align** 属性会使当前元素根据其余的 `inline` 元素进行垂直定位。

到底元素在垂直方向上处于什么样的位置取决于在同一行的其余元素，或 `line-height` 属性。

## 图示 ##
下面是一个图示以及部分的解释文本，来协助掌握，在 `inline` 元素上使用垂直定位时到底发生了什么：

![vertical-align](http://cdn.impressivewebs.com/2011-12/vertical-align-pic.jpg)

这里是一个 JSBin ，包含一些 `inline` 元素，其中的一个被置顶显示：
[JSBin](http://jsbin.com/isuvob/edit#html,live "Vertical Align")

## 关键字取值 ##

**vertical-align** 属性可以被设置为以下关键字：

- baseline （默认值）
- bottom
- middle
- sub
- super
- text-bottom
- text-top
- top

你可能只会使用其中的小部分，但是了解所有可选项总是好的。比如说在 [Demo](http://jsbin.com/isuvob/edit#html,live "Demo") 中，因为输入框的 `vertical-align` 属性被设置为 `top`，它根据最高的元素（较大的图片）置顶显示。

但是如果你不想根据图片或者其余的块状的元素进行定位，你可以选择 `text-top` 或者 `text-bottom`， 这样元素就会根据同一行的文本进行定位。

## 关于关键字 Middle ##

悲剧的是：**vertical-align: middle** 并不会像大家都期待的那样按照行内最高的元素进行居中定位，如果赋值为 **middle** 会使当前元素根据假象的小写 “x” （通常被称作 “x-height”） 进行居中定位。所以，根据它的实际作用， **middle** 改为 text-middle 会更合适一点。

在 [Demo](http://jsbin.com/apiqog/edit#html,live) 中，我把字体大小调大，从而让 “x-height” 变得很大，所以一般来说，不应该大量使用 **middle**。

## 非关键字取值 ##

 **vertical-align** 还可以取值为 `长度` 以及 `百分比`。以下的例子中的使用都是合法的：

----------

```
input {
        vertical-align: 100px;
}

span {
        vertical-align: 50%;
}

img {
        vertical-align: -300px;
}
```

虽然你可以阅读 [W3C关于 **vertical-align** 取值的解释](http://www.w3.org/TR/CSS21/visudet.html#propdef-vertical-align "W3C关于 **vertical-align** 取值的解释")，但是我觉得，自己尝试调整取值并查看其中的差异会更加有用。

## 总结 ##
一句话概括怎么来使用这个一直被误解的属性：

**vertical-align** 属性只在 `inline`、`inline-block` 元素以及单元格内容有效，对于单元格自身无效，它只会影响当前元素的对齐，而不是它的内容。

**PS：** 应该说明是，有些时候当我说 `inline` 元素的时候，`inline-block` 元素是包含在内的。

