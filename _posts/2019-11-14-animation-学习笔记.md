## css3 动画学习笔记

## animation

所谓动画，不过是让一个元素从一种 CSS 样式，变为另一种，这么一个过程，当然也不仅限于一种样式，播放动画时可以变化很多次，本节主要记录一些 CSS3 的 animation 属性用法和例子。

### 语法

语法上和其他属性是一样的，animation 属性是一个简写的属性，包含以下六个：

| 值                        | 描述                     |
| ------------------------- | ------------------------ |
| animation-name            | keyframe 名字            |
| animation-duration        | 完成一次动画的时间       |
| animation-timing-function | 动画的速度曲线           |
| animation-delay           | 延迟多少时间开始         |
| animation-iteration-count | 重复播放的次数           |
| animation-direction       | 是否应该轮流反向播放动画 |

> animation-duration 必须大于 0，否则播放时间为 0 就没有动画效果了

#### @keyframe

俗称关键帧，可以用这个属性来创建动画，语法：`@keyframe name {selector {css}}`，其中的 name 表明了帧的名字，需要和 animation-name 保持一致，选择器是过程的百分比，通常可以是`0%~100%, from(等同于0%), to(等同于100%)`，例子：

```css
@keyframes mymove {
  0% {
    top: 0px;
    left: 0px;
    background: red;
  }
  25% {
    top: 0px;
    left: 100px;
    background: blue;
  }
  50% {
    top: 100px;
    left: 100px;
    background: yellow;
  }
  75% {
    top: 100px;
    left: 0px;
    background: green;
  }
  100% {
    top: 0px;
    left: 0px;
    background: red;
  }
}
```

#### animation-delay

这个属性可以取负值，属性规定了初次播放动画时的延迟，默认是 0，当取负数时间时，比如-2s，表示跳过了前 2s 的动画，直接从第 2s 开始播放动画，这只对初始播放有效果。

#### animation-direction

规定是否交替播放动画，还可以规定初始是否反向；可选值如下表：

| 值                | 描述                             |
| ----------------- | -------------------------------- |
| normal            | 默认，正常播放                   |
| reverse           | 反向播放                         |
| alternate         | 正向反向交替播放，正向播放先出现 |
| alternate-reverse | 反向正向交替播放，反向播放先出现 |

#### animation-timing-function

规定动画播放的速度曲线，常规取值有：`linear, ease, ease-in, ease-out, ease-in-out`之外，还有一个是贝塞尔曲线，使用`cubic-bezier(n, n, n, n)`来规定动画播放的速度曲线，这是个很有意思的东西，[贝塞尔曲线在线](https://cubic-bezier.com/)，可以使用这个在线工具来生成曲线函数。

#### animation-fill-mode

规定了动画不播放时，可能是延迟播放，或者是已经播放完的这段时间内，应用的元素样式，可选值如下：

| 值        | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| none      | 默认，动画执行前后不会应用样式到目标元素，就是元素初始的样式 |
| forwards  | 动画播放结束后，元素会应用到动画最后的属性                   |
| backwards | 在动画开始时的延迟时间内，元素会应用`@key-frame`中的属性     |
| both      | 结合上述两个属性                                             |

有个例子：

```css
div {
  width: 100px;
  height: 100px;
  background: red;
  position: relative;
  animation: mymove 5s 1;
  animation-delay: 2s;
  animation-fill-mode: both;

  /*Safari and Chrome*/
  -webkit-animation: mymove 5s 1;
  -webkit-animation-delay: 2s;
  -webkit-animation-fill-mode: both;
}

@keyframes mymove {
  from {
    background: blue;
    left: 0px;
  }
  to {
    background: green;
    left: 200px;
  }
}

@-webkit-keyframes mymove /*Safari and Chrome*/ {
  from {
    background: blue;
    left: 0px;
  }
  to {
    left: 200px;
  }
}
```

## transition

transition 可以给元素添加过渡，从一种样式到另一种样式，也可以说是一种动画效果。

### 语法

看到的例子基本都是给元素设置伪类:hover 的时候，用到了过渡属性，属性如下表：

| 属性                       | 描述                                             |
| -------------------------- | ------------------------------------------------ |
| transition                 | 简写，在一个属性内设置以下四个属性               |
| transition-property        | 规定需要过渡的 css 属性名，如 width、height 这些 |
| transition-duration        | 过渡的持续时间                                   |
| transition-timing-function | 过渡效果的时间曲线                               |
| transition-delay           | 开始播放过渡的延迟                               |

#### transition-property

过渡的属性名，语法：`transition-property: none|all| property;`，默认是`all`，就是对 hover 内所有属性都使用过渡效果，也可以自定义一些属性加上过渡效果。

一个例子：

```html
<div class="dropdown">
  <div>鼠标移动到我这！</div>
  <div class="dropdown-content">
    <p>菜鸟教程</p>
    <p>www.runoob.com</p>
  </div>
</div>
<style>
  .dropdown {
    position: relative;
    display: inline-block;
  }
  .dropdown-content {
    position: absolute;
    background-color: #f9f9f9;
    min-width: 160px;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
    padding: 12px 16px;
    visibility: hidden;
    opacity: 0;
    -webkit-transition: 1200ms ease;
    transition: 1200ms ease;
  }
  .dropdown:hover .dropdown-content {
    visibility: visible;
    opacity: 1;
  }
</style>
```

## transform

transform 规定了元素的转换效果，分为 2D 和 3D 转换，可以实现对元素的移动，缩放，转动，拉伸等效果。

除`transform`以外的其他属性：
- `transform-origin`：`transform-origin: x-axis y-axis z-axis;`，用于更改转换元素的位置，需要在`transform`后定义，按照我个人理解，这个属性定义了旋转时以那个点为旋转中心，都设置为50%时，就是以中心点为旋转中心，[MDN: transform-origin](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin)
- `transform-style`：规定了嵌套的元素是否保留3D属性，`transform-style: flat|preserve-3d;`默认值为`flat`，不保留；
- `backface-visibility`：规定了元素背面是否可见，可选值有`visible | hidden`

### 2D转换方法

- `translate`：`transform: translate(xAxis, yAxis);`，用于将元素从当前位置移动到偏移量规定的位置；
- `rotate`：`transform: rotate(angle);`，用于旋转元素；
- `scale`：`transform: scale(xAxis, yAxis);`，用于缩放元素；
- `skew`：`transform:skew(<angle> [,<angle>]);`，用于倾斜元素；

以上方法除了最后一个是属性外，其他方法设计到X,Y轴的都可以用`methodX(), methodY()`单独对某条轴设置。

### 3D转换方法

上述的2D方法中，名称改为`method3d`，如`translate3D`，并添加Z轴参数。

## 总结

本文主要介绍了一些CSS3常用的动画属性，包括了`animation | transition | transform`，这几个属性结合也可以获得一些特殊的动画效果。

最后丢一个应用的例子，写了一个仿照antd的loading组件，直接用`vue-cli 3.0`的命令`vue serve`就能看到效果了；

```html
<!--
 * @Description: 一个loading的组件，仿antd
 * @Author: youzi
 * @Date: 2019-11-29 09:04:27
 * @LastEditors: youzi
 * @LastEditTime: 2019-11-29 16:14:06
 -->

<template>
  <div class="loading-container">
    <div class="dot">
      <span class="left-top"></span>
      <span class="right-top"></span>
      <span class="left-bottom"></span>
      <span class="right-bottom"></span>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Loading'
};
</script>
<style lang="less" scoped>
.loading-container {
  z-index: 99;
  background-color: #e6f7ff;
}
.dot {
  @dot-size: 30px;
  display: flex;
  justify-content: center;
  align-content: center;
  width: @dot-size;
  height: @dot-size;
  position: relative;
  animation: rotate 1.2s infinite linear;
  border-radius: 50%;
  @keyframes rotate {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  span {
    display: block;
    position: absolute;
    @i-size: 12px;
    @i-color: rgba(26, 207, 108, 0.623);
    width: @i-size;
    height: @i-size;
    border-radius: 50%;
    background-color: @i-color;
    animation: opacity 1s infinite linear alternate;
  }
  @keyframes opacity {
    0% {
      opacity: 0.3;
    }
    100% {
      opacity: 1;
    }
  }
  .left-top {
    top: 0;
    left: 0;
    transform: rotate(-45deg);
    animation-delay: 0s;
  }
  .right-top {
    top: 0;
    right: 0;
    transform: rotate(45deg);
    animation-delay: 0.4s;
  }
  .left-bottom {
    left: 0;
    bottom: 0;
    transform: rotate(45deg);
    animation-delay: 0.8s;
  }
  .right-bottom {
    right: 0;
    bottom: 0;
    transform: rotate(-45deg);
    animation-delay: 1.2s;
  }
}
</style>
```