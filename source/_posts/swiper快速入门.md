---
title: swiper快速入门
date: 2020-04-21 21:30:49
tags: vue
---

>swiper触摸滑动插件，效果帅不帅？可以看苹果官网，那些神奇的延迟过渡效果
>
>这个组件入门很快，官网主要针对原生JS使用，不过有对vue单开了一个文档，只不过是在github里，并且只有DEMO，没有细讲vue内参数使用，不过这些在针对原生JS文档里都有讲到，用起来也差不多。
>
>下面提供文档和一个DEMO解析

<!-- more -->


#### 安装

```cmd
npm install swiper vue-awesome-swiper --save

# or
yarn add swiper vue-awesome-swiper
```

> https://github.com/surmon-china/vue-awesome-swiper

#### 快速入门

swiper 容器

swiper-slide 节点

:options="配置"

```html
<template>
  <swiper class="swiper" :options="swiperOption">
    <swiper-slide>Slide 1</swiper-slide>
    <swiper-slide>Slide 2</swiper-slide>
    <swiper-slide>Slide 3</swiper-slide>
  </swiper>
</template>
```

#### vue用户使用swiper官方文档

```json
options:{

​	下面的属性

}
```

![](/img/swiper/1.png)

如果属性类型为Object

```json
options:{

    keyboard:{

    	enabled: true,

    	onlyInViewport: true,

    }

}
```

![](/img/swiper/2.png)

下面展示demo

#### 效果

![](/img/swiper/GIF.gif)

#### DOM

```html
<template>
  <swiper class="swiper" :options="swiperOption">
    <!-- 
      slot="parallax-bg" 背景插槽
      data-swiper-parallax="-20%" 从左到右 背景图片最多偏移的百分比
     -->
    <div class="parallax-bg" slot="parallax-bg" data-swiper-parallax="-20%"></div>
    <swiper-slide>
      <!-- 内容，每一页的内容，随便写 -->
      <div class="title" data-swiper-parallax="-100">Slide 1</div>
      <div class="subtitle" data-swiper-parallax="-240">Subtitle</div>
      <div class="text" data-swiper-parallax="-360">
        <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam dictum mattis velit, sit amet faucibus felis iaculis nec. Nulla laoreet justo vitae porttitor porttitor. Suspendisse in sem justo. Integer laoreet magna nec elit suscipit, ac laoreet nibh euismod. Aliquam hendrerit lorem at elit facilisis rutrum. Ut at ullamcorper velit. Nulla ligula nisi, imperdiet ut lacinia nec, tincidunt ut libero. Aenean feugiat non eros quis feugiat.</p>
      </div>
    </swiper-slide>
    <swiper-slide>
      <div class="title" data-swiper-parallax="-100">Slide 2</div>
      <div class="subtitle" data-swiper-parallax="-240">Subtitle</div>
      <div class="text" data-swiper-parallax="-360">
        <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam dictum mattis velit, sit amet faucibus felis iaculis nec. Nulla laoreet justo vitae porttitor porttitor. Suspendisse in sem justo. Integer laoreet magna nec elit suscipit, ac laoreet nibh euismod. Aliquam hendrerit lorem at elit facilisis rutrum. Ut at ullamcorper velit. Nulla ligula nisi, imperdiet ut lacinia nec, tincidunt ut libero. Aenean feugiat non eros quis feugiat.</p>
      </div>
    </swiper-slide>
    <swiper-slide>
      <div class="title" data-swiper-parallax="-100">Slide 3</div>
      <div class="subtitle" data-swiper-parallax="-240">Subtitle</div>
      <div class="text" data-swiper-parallax="-360">
        <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam dictum mattis velit, sit amet faucibus felis iaculis nec. Nulla laoreet justo vitae porttitor porttitor. Suspendisse in sem justo. Integer laoreet magna nec elit suscipit, ac laoreet nibh euismod. Aliquam hendrerit lorem at elit facilisis rutrum. Ut at ullamcorper velit. Nulla ligula nisi, imperdiet ut lacinia nec, tincidunt ut libero. Aenean feugiat non eros quis feugiat.</p>
      </div>
    </swiper-slide>
    <!-- 
      pagination 滚动条
      button-prev 上一页 < 
      button-next 下一页 >
     -->
    <div class="swiper-pagination swiper-pagination-white" slot="pagination"></div>
    <div class="swiper-button-prev swiper-button-white" slot="button-prev"></div>
    <div class="swiper-button-next swiper-button-white" slot="button-next"></div>
  </swiper>
</template>
```

swiper-slide 里的不重要，显示的字好看些而已

#### JS

```js
<script>
import { Swiper, SwiperSlide } from "vue-awesome-swiper";
import "swiper/css/swiper.css";
export default {
  name: "swiper-example-parallax",
  title: "Parallax",
  components: {
    Swiper,
    SwiperSlide
  },
  data() {
    return {
      swiperOption: {
        speed: 600, // 过渡速度
        parallax: true, //开启视差效果
        pagination: {
          el: ".swiper-pagination", // 下面的点点 ... 绑定class
          clickable: true //点点 可以点击跳转
        },
        navigation: {
          nextEl: ".swiper-button-next", //下一页 > 绑定class
          prevEl: ".swiper-button-prev"  //上一页 < 绑定class
        }
      }
    };
  }
};
</script>
```

swiper :option绑定这个swiperOption

#### CSS
```less
<style lang="less" scoped>
.parallax-bg {
  position: absolute;
  left: 0;
  top: 0;
  width: 130%;
  height: 100%;
  background-size: cover;
  background-position: left;
  background-image: url("../../../assets/img/2.jpg"); //指向自己的图片，背景图
}
.swiper {
  width: 100%;
  height: 380px;
  .swiper-slide {
    display: flex;
    flex-direction: column;
    justify-content: center;
    color: white;
    box-sizing: border-box; //边缘适应
    padding: 0 80px;
    background-color: transparent;
    .title {
      font-weight: bold;
    }
    .subtitle {
      text-align: left;
      margin-left: 50px;
      margin-bottom: 30px;
      font-size: 30px;
    }
    .text {
      max-width: 430px;
      line-height: 1.32;
    }
  }
}
</style>
```
使用less 的 嵌套式css，使用前安装less。`https://less.bootcss.com/`

#### 参考

```
https://segmentfault.com/a/1190000014609379  个人用户整理文档
https://www.swiper.com.cn/ 官方文档
https://github.com/surmon-china/vue-awesome-swiper 针对VUE的文档
https://github.surmon.me/vue-awesome-swiper/ 针对VUE的DEMO
```









