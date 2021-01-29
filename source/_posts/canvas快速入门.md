---
title: canvas快速入门
date: 2020-04-11 16:34:48
tags: js
---
> canvas做动画的效果比div流畅很多，而且复杂的动画不会很卡，很多效果是由js实现的，css可以提供的属性比较少，canvas做一些牛皮的动画，更多的需要一点几何学（滑稽）
>
> canvas做动画的原理是一帧一帧来的（可以体会到画家的辛苦哦），每次需要清空画板重新渲染，所以在做动画需要一直循环然后通过公式来绘制出运动轨迹。
>
> 下面我会放其他教程更全面的链接 和 一个Demo

<!--more-->

#### 效果

![](/img/md3/GIF.gif)

#### DOM节点

```html
<canvas width="400" height="200" style="border: 1px solid #000000;">
    你的浏览器不支持canvas，请升级浏览器
</canvas>
```

width和height只能在canvas节点中设置

如果canvas渲染失败，会显示内容

#### 初始2d画布

```js
var canvas = document.getElementsByTagName('canvas')[0] //获取节点
var ctx = canvas.getContext('2d') //得到2d画板
```

有多个画布的话 getTag可以更方便的获取想操作的画布

填`3d`就得到3d画板了

#### 矩形构造函数

```js
function ItcastRect(option) {
    //矩形构造函数
    this._init(option);
}
ItcastRect.prototype = {
    //矩形的原型对象
    _init: function(option) {
        //初始化方法
        option = option || {};
        this.x = option.x === 0 ? 0 : option.x || 100;
        this.y = option.y === 0 ? 0 : option.y || 100;
        this.w = option.w || 100;
        this.h = option.h || 100;
        this.angle = option.angle === 0 ? 0 : option.angle || 0;
        this.fillStyle = option.fillStyle || "silver";
        this.strokeStyle = option.strokeStyle || "red";
        this.strokeWidth = option.strokeWidth || 4;
        this.scaleX = option.scaleX || 1;
        this.scaleY = option.Y || 1;
    },
    render: function(ctx) {
        ctx.clearRect(0, 0, 400, 200); //清屏 从0,0开始大小400*200
        //把矩形渲染到canvas中
        ctx.save();
        ctx.translate(this.x, this.y); //位移画布
        ctx.rotate((this.angle * Math.PI) / 180); //旋转角度
        ctx.scale(this.scaleX, this.scaleY); //缩放
        ctx.fillStyle = this.fillStyle;
        ctx.fillRect(0, 0, this.w, this.h); //填充矩形
        ctx.lineWidth = this.strokeWidth; //线宽
        ctx.strokeStyle = this.strokeStyle; //填充样式
        ctx.strokeRect(0, 0, this.w, this.h); //描边样式
        ctx.restore();
    },
    constructor: ItcastRect
};
```

这里构造函数new出来后就是一个方块，并且每次render都会清屏一次，清屏大小是拟定的，跟canvas大小一样

constructor可以不写，这里重定向原型到`ItcastRect`方法上，不写的话默认返回Object

#### 移动方块

```js
//初始化一下方块
const rect = new ItcastRect();
let x = 0, //初始位置
    y = 50,
    speed = 100, // 100px/1s 每秒移动一百像素
    fps = 60, // 帧率
    x_step = speed / fps, //每次移动像素
    y_step = speed / fps;
/**
  * 移动
  * @param {Object} rect - 方块实例
  */
function move(rect) {
    rect._init({
        x,
        y
    });
    rect.render(ctx);
    x += x_step;
    y += y_step;
    if (x >= 300 || x < 0) {
        x_step *= -1;
    }
    if (y >= 100 || y < 0) {
        y_step *= -1;
    }
    setTimeout(() => {
        move(rect);
    }, 1000 / fps);
}
move(rect);
```

用递归的方式移动方块，用一些简单的公式计算下一个位置，然后清空画布重新渲染。

### canvas详解

>  https://malun666.github.io/aicoder_vip_doc/#/pages/canvas

讲的挺好的，canvas有很多基础属性需要花点时间看看。

