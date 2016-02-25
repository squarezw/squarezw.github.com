---
layout: post
title: "FlexBox 使用向导"
description: "a guide to flexbox"
category: 
tags: [flex layout box]
---
## 简介

    Flexbox 布局模块提供一个容器，使其中的元素，可以采用更便捷的方法实现布局、对齐、分布间距等方式。甚至当大小不定时，可以自动伸缩（它们称为"flex"）

    它的思想主要是基于 flex layout，调整容器里的元素宽/高，填充合适空格（更多用于不同设备的屏幕的适配）。

## 基本述语

**在了解 Flexbox 述语前，有必要先看看 CSS 中关于 [inline-block](http://www.w3schools.com/css/css_inline-block.asp) 的介绍**

Flexbox 是一个完整的模块，不是一个单一属性，它包含了很多设置属性。 它们中的一些被设置为容器(父元素，被称为 "flex container")，其它一些被设置为子元素（称为 "flex items"）

对于规则的布局是基于两块之间的相对方向流线性布局，这个 flex layout 是基于 "flex-flow 方向"。 请看下图以更好解释它。

![Flexbox述语](/assets/flexbox.png)

基本上它会基于主轴(从main-start 到 main-end)或交叉轴（从cross-start 到 cross-end）方向排列它们。

- main axis flex容器的主轴主要用于排列它的元素。不过要注意，它不一定是水平方向，它依赖于 flex-direction 属性 (见下面说明)
- main-start | main-end flex 容器里的元素放置从main-start 开始到 main-end 结束
- main-size flex 元素的宽或高，有 width 与 height 属性
- cross axis 与主轴相交的即为交叉轴。它的方向依赖与主轴方向
- cross-start | cross-end Flex 基于 cross-start 与 cross-end 排列它的元素
- cross size 一个 flex 元素的宽或高

## 容器

![Flexbox 容器](/assets/flex-container.svg)

容器属性

### display

定义一个 flex 容器，给定值指定是内联(inline)或者块(block)。它可以打开一个 flex 环境用来决定元素的填充方式。

```
.container {
    display: flex; /* 或 inline-flex */
}
```

### flex-direction

![Flex 方向](/assets/flex-direction1.svg)

决定主轴方向

```
.container {
    flex-direction: row | row-reverse | column | column-reverse;
}
```

- row (默认): 从左到右
- row-reverse: 从右到左
- column: 从上到下
- column-reverse: 从下到上

### flex-wrap

![Flex Wrap](/assets/flex-wrap.svg)

默认情况下 flex 元素会在一行显示. 你可以更变属性决定折行与折行的方向

```
.container {
    flex-wrap: nowrap | wrap | wrap-reverse;
}
```

- nowrap (默认): 单行
- wrap: 折行
- wrap-reverse: 反向折行

### justify-content

![内容对齐](/assets/justify-content.svg)

决定如何在主轴上分布空白区域

```
.container {
    justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

- flex-start (默认)：基于 start line 排列
- flex-end: 基于 end line 排列
- center: 居中
- space-between: 均匀的分布间距，第一个元素在首行，最后一个在未行
- space-around: 均匀的分布空白区域在各元素周围

### align-items

![内容对齐](/assets/align-items.svg)

决定交叉轴的排列方式

```
.container {
    align-items: flex-start | flex-end | center | baseline | stretch;
}
```

- flex-start
- flex-end
- center
- baseline
- stretch (默认)

### align-content

![排列内容](/assets/align-content.svg)

类似于 justify-content ，交叉轴的内容排列

-- 如果只有一行时没有效果

```
.container {
    align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

- flex-start
- flex-end
- center
- space-between
- space-around
- stretch (默认)

## 元素

![flex 元素](/assets/flex-items.svg)

元素属性

### order

默认情况下，flex 元素，按来源顺序排列和个元素，你也可以分别指定元素的 order 属性来重新排列出现的顺序。

```
.item {
    order: <integer>;
}
```

### flex-grow

![flex 伸缩](/assets/flex-grow.svg)

决定各元素的你伸长比例，它是一个无符号单位。如果所有元素都设置为 1 ，它们将会是同一大小的排列在容器里。如果你给定一个值是2，那他会针对其他对象的两倍。

```
.item {
  flex-grow: <number>; /* 默认是 0 */
}
```

### flex-basis

```
.item {
  flex-basis: <length> | auto; /* default auto */
}
```

[看这张图](http://www.w3.org/TR/css3-flexbox/images/rel-vs-abs-flex.svg)

### flex

快捷方式 flex-grow, flex-shrink and flex-basis 这三个属性。第二和第三个参数为可选, 默认为 0 1 auto

```
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

### align-self

![自我排列](/assets/align-self.svg)

可以指定某个元素的排列行为，请参考 align-items 解释

```
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

### 示例

解决一个经常会遇到的问题：完美居中

```
.parent {
  display: flex;
  height: 300px; /* Or whatever */
}

.child {
  width: 100px;  /* Or whatever */
  height: 100px; /* Or whatever */
  margin: auto;  /* Magic! */
}
```

---

## 其它相关资源

[Facebook Css Layout](https://github.com/facebook/css-layout)

[Flexbox in the CSS specifications](http://www.w3.org/TR/css3-flexbox/)

[Flexplorer by Bennett Feely](http://bennettfeely.com/flexplorer/)

[Flexbox at MDN](https://developer.mozilla.org/en-US/docs/CSS/Tutorials/Using_CSS_flexible_boxes)

[Flexbox at Opera](http://dev.opera.com/articles/view/flexbox-basics/)

[Diving into Flexbox by Bocoup](http://weblog.bocoup.com/dive-into-flexbox/)

[Mixing syntaxes for best browser support on CSS-Tricks](https://css-tricks.com/using-flexbox/)

[Flexbox by Raphael Goetter (FR)](http://www.alsacreations.com/tuto/lire/1493-css3-flexbox-layout-module.html)

{% include JB/setup %}
