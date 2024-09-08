---
layout: post
title: 'react 入门'
date: 2022-12-29 20:31:42
author: 'Youzi'
catalog: true
tags:
  - react
---

# CSS滚动条的特性

移动端总有这些滚动条的特殊样式，刚好晚上写了一个，现在总结一下。

## webkit内核浏览器特有属性

![webkit-scrollbar](/img/in-post/fscroll-bar/scroll.png)

- ::-webkit-scrollbar——整个滚动条。
- ::-webkit-scrollbar-button——滚动条上的按钮（上下箭头）。
- ::-webkit-scrollbar-thumb——滚动条上的滚动滑块。
- ::-webkit-scrollbar-track——滚动条轨道。
- ::-webkit-scrollbar-track-piece——滚动条没有滑块的轨道部分。
- ::-webkit-scrollbar-corner——当同时有垂直滚动条和水平滚动条时交汇的部分。通常是浏览器窗口的右下角。
- ::-webkit-resizer——出现在某些元素底角的可拖动调整大小的滑块。

### `-webkit-scrollbar`

整个滚动条的样式配置

```css
::-webkit-scrollbar {
  width: 10px; /* 纵向滚动条的宽度 */
  height: 10px; /* 横向滚动条的高度 */
  background: rgb(18, 209, 104); /* 背景 */
  border-radius: 10px; /* 圆角 */
}
```

### `-webkit-scrollbar-button`

滚动条两端，上下两个按钮的样式

```css
::-webkit-scrollbar-button {
  width: 10px; /* 横向滚动条 宽度 */
  height: 10px; /* 纵向滚动条 高度 */
  background: black;
  border-radius: 10px;
}
```

### `-webkit-scrollbar-track`

外部轨道，除了内轨道和滑块的其余部分；

```css
::-webkit-scrollbar-track {
  background: red;
  border-radius: 10px;
}
```

### `-webkit-scrollbar-track-piece`

内轨道，供滑块滑动的轨道部分，会影响或者覆盖外轨道样式；

**特别注意`margin`属性，会影响轨道长度，值越大，则内轨道长度被压缩的越小**

```css
::-webkit-scrollbar-track-piece {
  background-color: blue;
  margin: 20px; /* 这个属性会影响内轨道的长度 */
  border-radius: 10px; 
}
```

### `-webkit-scrollbar-thumb`

滑块样式，主要可以配置背景颜色和圆角

```css
::-webkit-scrollbar-thumb {
  background: pink;
  border-radius: 10px;
}
```

### `-webkit-scrollbar-corner`

当同时有垂直滚动条和水平滚动条时交汇的部分。通常是浏览器窗口的右下角。

```css
::-webkit-scrollbar-corner {
  min-height: 20px; /* 最小高度或者宽度 */
  min-width: 20px;
  background: brown;
}
```

### `:hover`

上面的css属性都可以加上伪类，鼠标悬浮的样式。
