---
title: 前端页面动画实现方式
tags: []
id: '72'
categories:
  - - 前端
date: 2020-10-29 19:04:14
---

![](../images/uploads/sites/2/2020/10/images.jpeg)

> 实现方式： 方式一：css3 transitions属性方法 方式二：css3 animations属性方法 方式三：web animations API

#### transitions实现

**基本使用方法**

```css
transition:property duration timing-function
```

```
*property-过渡属性
duration-过渡时长
timing-function-过渡方法*
```

**平滑过渡多个属性** 使用','分割多个过渡属性，示例如下：

```css
.transitions {
    background-color: #a9ce07;
    transform: rotate(45deg) translate(80px, 100px) scale(1.5);
   ** transition: background-color 1s linear, color 1s linear, transform 1s linear; **
}

.transitions:hover {
    background-color: #ff0000;
    color: #00ff00;
    transform: rotate(0) translate(0, 0) scale(1);
}
```

#### animations实现

不同于transitions只能通过指定属性开始值与结束值的动画实现，animations动画功能引入了关键帧的概念，通过多个关键帧可以实现更为复杂的动画效果。 **使用示例**

```css
@keyframes mycolor {
    0% {
        background-color: #a9ce07;
    }
    40% {
        background-color: #ff0000;
    }
    70% {
        background-color: #ffff00;
    }
    100% {
        background-color: #0000ff;
    }
}

.animations:hover {
    animation: mycolor 1s linear;
}
```

示例说明： （1）先编写关键帧集合，其中`mycolor`为关键帧集合名称 （2）关键帧代码如下

```css
40%{
    样式代码
}
```

其中40%表示该帧处于动画工程中的40%处,括号内部书写关键帧需要改变的样式代码。 （3）创建好关键帧集合后，在元素的样式中通过animation属性使用该关键帧集合。 **animation相关属性介绍** animation-name：指定关键帧集合名称 animation-duration：动画所花费的时长 animation-timing-function：动画方法 animation-delay：动画延迟时间 animation-iteration-count：动画执行次数 animation-direction：动画执行方向 **一行样式书写方式如下**

```css
animation:name duration ti ming-function delay iteration-count-diretion;
```

#### Web Animations API

web animations api是animations和JavaScript的结合体，可以使用JavaScript控制元素，具有和css一样的性能。 **使用方法** （1）定义关键帧

```javascript
let keyframes=[
    {
        transform:'translateX(0px) translateY(0px)'
    },{
        transform:'translateX(300px) translateY(0px)'
    },{
        transform:'translateX(300px) translateY(300px)'
    }
]
```

说明： web Animations API 默认平均分配动画进程，若需显示定义某个关键帧的出现时刻，需要用到一个`offset`属性，属性值为一个0-1的小数点值，示例如下：

```javascript
{
    transform:'translateX(0px) translateY(0px)',
        offset:0.2
}
```

若关键帧数组只有一个关键帧对象，浏览器将抛出一个NotSupportedErrorcuowu （2）设置动画相关选项，示例如下：

```javascript
let set={
    duration:3000,
    iterations:Infinity
}
```

**其他相关选项** id-动画标识符 delay-动画延迟时间 direction-动画执行方向 duration-动画花费时长 easing-指定动画方法 iterations-动画执行次数，可设置Infinity（无限次） （3）js执行动画 使用dom对象的animate方法执行动画，示例如下：

```javascript
elem.animate(keyFrames,set)
```